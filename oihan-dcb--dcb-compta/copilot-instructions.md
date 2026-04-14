## dcb-compta

> DCB Compta est une application comptable interne pour Destination Cote Basque (conciergerie ~50 biens, Biarritz).

# Instructions pour Claude Code — DCB Compta

## Contexte projet
DCB Compta est une application comptable interne pour Destination Cote Basque (conciergerie ~50 biens, Biarritz).
Stack : React + Vite, Supabase (Postgres + Edge Functions Deno), Vercel, GitHub (org: Oihan-dcb).

## Documentation — lire au demarrage
Les fichiers dans `docs/` sont la source de verite du projet. Lis-les avec `/init` ou manuellement.

| Fichier | Contenu |
|||
| `docs/project-overview.md` | Vue d'ensemble, stack, bugs connus, historique fixes |
| `docs/architecture-map.md` | Carte des modules, flux de donnees, dependances |
| `docs/data-model.md` | Schema Supabase complet, champs, contraintes |
| `docs/domain-rules.md` | Regles metier : ventilation, facturation, AUTO, HAOWNER |
| `docs/invariants.md` | Invariants systeme avec statut ✅/⚠/❌ |
| `docs/source-of-truth.md` | Sources de donnees, priorites, comportements |

## Regle obligatoire — Mise a jour docs/
Apres chaque fix ou feature significatif, mets a jour le(s) fichier(s) docs/ concerne(s) :
- Bug corrige : passer ❌ → ✅ dans `invariants.md`
- Nouveau comportement : documenter dans `domain-rules.md`
- Schema modifie : mettre a jour `data-model.md`
- Architecture changee : mettre a jour `architecture-map.md`
- Fix important : ajouter dans la section "Fixes" de `project-overview.md`

## Regles techniques
- Toujours `npm run build` avant de pousser
- Commits atomiques avec message explicite
- Ne jamais utiliser `btoa(unescape(encodeURIComponent()))` pour encoder des fichiers
- Pour encoder un fichier source via l'API GitHub, utiliser uniquement :
  ```js
  const blob = new Blob([src], { type: 'text/plain;charset=utf-8' })
  const ab = await blob.arrayBuffer()
  const u8 = new Uint8Array(ab)
  let b64 = ''
  for (let i = 0; i < u8.length; i += 3072) b64 += btoa(String.fromCharCode(...u8.slice(i, i+3072)))
  ```

## Codes comptables
HON (honoraires DCB), FMEN (forfait menage), AUTO (debours AE), LOY (reversement proprietaire),
VIR (virement reel), TAXE (taxe de sejour), DEB_AE (debours facture separee), HAOWNER (achat DCB pour proprio), PREST (memo prestation)

## Charte graphique (zero bleu)
- `--brand #CC9933` or, `--bg #F7F3EC` creme, `--border #D9CEB8`, `--text #2C2416` brun
- Header nav `#EAE3D4` + filet or 2px

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Oihan-dcb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Oihan-dcb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
