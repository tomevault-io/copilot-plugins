## skyrimck-mcp

> Serveur MCP (Model Context Protocol) pour manipuler des fichiers ESP/ESM de Skyrim SE sans Creation Kit. Permet a Claude Code de creer des quetes, scenes, packages IA, aliases, etc. directement.

# CLAUDE.md — SkyrimCK-MCP

Serveur MCP (Model Context Protocol) pour manipuler des fichiers ESP/ESM de Skyrim SE sans Creation Kit. Permet a Claude Code de creer des quetes, scenes, packages IA, aliases, etc. directement.

## Objectif

Rendre le Creation Kit accessible aux aveugles en le remplacant par Claude Code + ce serveur MCP. Au lieu d'utiliser l'interface graphique du CK, l'utilisateur decrit ce qu'il veut en francais et Claude genere l'ESP.

## Cas d'usage concret (premier objectif)

Generer l'ESP pour l'autowalk du projet SkyrimNVDA :
- 1 quete (`SkyrimTTS_AutoWalkQuest`, Start Game Enabled)
- 2 aliases (DstMarker vide, Traveler = PlayerRef)
- 1 scene (`SkyrimTTS_WalkScene`, Override Behavior, Repeat)
- 1 package Travel (vers DstMarker, vitesse course, nage autorisee)
- 1 script Papyrus attache

Le doc de spec complet est dans : `c:\Users\marcd\source\repos\SkyrimNVDA\docs\autowalk_spec.md`

## Librairie choisie : esper

- **Repo** : https://github.com/matortheeternal/esper
- **Langage** : C# (.NET 8.0) + interop JavaScript
- **Auteur** : matortheeternal (createur de zEdit et Mator Smash)
- **Derniere mise a jour** : septembre 2025
- **Licence** : a verifier

### Pourquoi esper

C'est la seule librairie qui :
- Lit ET ecrit des fichiers ESP
- Supporte Skyrim SE explicitement (SSE.json dans definitions.zip)
- Gere tous les types de records necessaires : QUST, SCEN, PACK, ALST/ALLS
- Est maintenue et creee par un dev reconnu du modding
- Basee sur les definitions xEdit (couverture complete des records)

### Librairies rejetees
- **esp-parser** (Python) : Fallout 3/NV seulement, pas Skyrim
- **skytool** (Python) : abandonne 2013, lecture seule
- **bethesda-structs** (Python) : pas de support Skyrim
- **esplugin** (Rust) : lecture seule

## Architecture prevue

```
SkyrimCK-MCP/
├── CLAUDE.md          (ce fichier)
├── src/               Code du serveur MCP
├── esper/             Librairie esper (git submodule ou copie)
└── output/            ESP generes
```

### Comment ca marche

1. Le serveur MCP tourne en arriere-plan (Node.js ou C#)
2. Claude Code s'y connecte via le protocole MCP
3. Claude envoie des commandes : "cree une quete", "ajoute un alias", etc.
4. Le serveur utilise esper pour construire l'ESP en memoire
5. Le serveur sauvegarde l'ESP dans output/
6. L'utilisateur copie l'ESP dans Skyrim Data/

### Commandes MCP envisagees

- `create-plugin` : creer un nouveau fichier ESP vide
- `add-quest` : ajouter une quete avec ses proprietes
- `add-alias` : ajouter un alias a une quete
- `add-scene` : creer une scene
- `add-package` : creer un package IA
- `attach-script` : attacher un script Papyrus a un record
- `save-plugin` : sauvegarder l'ESP sur disque
- `list-records` : lister les records dans un ESP existant

## Etapes de developpement

### Phase 1 : Prototype
1. Cloner esper et verifier qu'il compile (.NET 8.0)
2. Ecrire un test simple : creer un ESP avec une quete vide
3. Verifier que l'ESP se charge dans Skyrim sans crash

### Phase 2 : Serveur MCP basique
4. Creer le serveur MCP (probablement en C# pour rester dans le meme ecosysteme qu'esper)
5. Implementer create-plugin, add-quest, save-plugin
6. Tester la connexion depuis Claude Code

### Phase 3 : Records complets
7. Implementer add-alias, add-scene, add-package
8. Generer l'ESP autowalk complet
9. Tester en jeu

### Phase 4 : Compilation Papyrus
10. Integrer Caprica (compilateur Papyrus standalone) pour compiler les .psc en .pex
11. Inclure les .pex dans l'ESP ou a cote

## Lecons apprises (pieges a eviter)

### Sons custom dans un ESP

Pour jouer des sons custom via Papyrus, il faut **DEUX types de records** :
1. **SNDR** (Sound Descriptor) : contient le chemin du .wav, le flag loop, la categorie, etc.
2. **SOUN** (Sound Marker) : wrapper obligatoire. C'est le type `Sound` en Papyrus.

Le type Papyrus `Sound` = SOUN, **PAS** SNDR. Si on fait `Game.GetFormFromFile(formID) as Sound` sur un SNDR, ca retourne None.

**Structure SOUN** (3 subrecords) :
- EDID : nom
- OBND : 12 bytes de zeros
- SDSC : FormID du SNDR associe

**Structure SNDR** (7 subrecords, dans cet ordre) :
- EDID, CNAM (`0A 54 EF 1E`), GNAM (`0x0007F80B`), ANAM (chemin wav), ONAM (`0x000ABEF3`), LNAM (looping), BNAM (priority)
- Les valeurs CNAM/GNAM/ONAM sont copiees de SkyrimAccessibility (mod de Dio) et fonctionnent
- LNAM byte[1] : 0=None, 8=Loop
- Chemin ANAM : `Data\Sound\fx\SkyrimTTS\fichier.wav`

**Fichiers WAV** : PCM 16 bits, 44100 Hz, mono ou stereo. Pas de WAVE_FORMAT_EXTENSIBLE ni 24 bits.

**Pour un son en boucle** : le fichier WAV doit etre court (~0.3s) sinon le delai entre repetitions est trop long.

### VMAD (scripts Papyrus)

Le VMAD pour une quete avec script + script sur alias :
- Section 1 : scripts de la quete (properties avec Object v2 format)
- Section 2 : quest fragments (unknown=2, fragmentCount=0, fileName vide)
- Section 3 : alias scripts (aliasID + VMAD imbrique avec scripts de l'alias)

### Packages Travel (PACK)

- PKDT General Flags : bit 13 = "Preferred Speed" (OBLIGATOIRE pour que la vitesse fonctionne), bit 18 = "Allow Swimming"
- CTDA conditions : GetSitting==0 (evite bug meubles), IsInDialogueWithPlayer==0, IsInScene==0
- ALPC subrecord dans QUST : attache le package a un alias directement (pas besoin de SCEN)

### Game.FastTravel()

`Game.FastTravel(ObjectReference)` est une fonction Papyrus **native vanilla** qui teleporte le joueur. Elle ne montre PAS de boite de confirmation (contrairement au clic sur la carte). La confirmation doit etre geree cote C++.

### Compilation Papyrus

```
"C:/Program Files (x86)/Steam/steamapps/common/Skyrim Special Edition 1946180/Papyrus Compiler/PapyrusCompiler.exe" script.psc -i="scripts;chemin\Scripts\Source;chemin\Source\Scripts" -o="scripts" -f="TESV_Papyrus_Flags.flg"
```

### Game.GetFormFromFile vs Game.GetForm

Toujours utiliser `Game.GetFormFromFile(localFormID, "plugin.esp")` (fonction SKSE) plutot que `Game.GetForm(runtimeFormID)`. Ca evite les problemes de load order.

## Prerequis

- .NET 8.0 SDK (pour compiler esper)
- Git (pour cloner esper)
- Skyrim SE installe (pour tester les ESP)
- Optionnel : Caprica pour la compilation Papyrus

## Projet parent

Ce projet est un outil pour le projet principal :
`c:\Users\marcd\source\repos\SkyrimNVDA\` (plugin SKSE d'accessibilite)

## Preferences utilisateur

- **Toujours parler en francais**
- **Vulgariser** : expliquer simplement, pas de jargon non explique
- L'utilisateur est aveugle, utilise NVDA
- GitHub username : Pyrhame

---
> Source: [Pyrhame/SkyrimCK-MCP](https://github.com/Pyrhame/SkyrimCK-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
