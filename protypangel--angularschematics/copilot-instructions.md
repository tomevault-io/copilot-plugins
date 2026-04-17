## angularschematics

> Project's structure, generation files and schematics definitionD

# Dossiers
## config
- SchematiqueName est en format classify donc chaque mot doit être en majuscule et coller les un à la suite des autres.
- collection.json : définition des schematiques.
- les schematiques dans collection.json, doivent contenir :
  - une description
  - un factory qui pointe toujours sur "../dist/index#[SchematiqueName]"
  - schema qui point sur "./Schema[SchematiqueName]" 
- Schema[SchematiqueName].json : description du schematique.
- Dossier de configuration de jasmine

## templates 
- Fichiers utilisé par les schematiques.
- Chaque schematique doit avoir son propre dossier dédié dans files.

## src
- /index.ts : s'occupe des appelles des schematique
- [SchematiqueName] : 
  - Schema[SchematiqueName].d.ts : interface des éléments du schematique, utilisé par le paramètre option de la fonction retournant le type Rule de "@angular-devkit/schematics"
  - S'occupe de la logique du son schematique.
  - Chaque fichier de logique doit avoir son propre fichier de declaration de type et d'interface 

## tests : 
- [SchematiqueName].spec.ts : s'occupe des testes d'intégration et unitaire, chaque describe doit s'occupe de sa propre parties de code. Il ne faut pas que ce fichier soit digeste à lire.
- [SchematiqueName].spec.util.ts : permettre de s'occuper de la parties logique utilisé par [SchematiqueName].spec.ts, pour que [SchematiqueName].spec.ts ne soit pas trop digeste.


# Infos supplémentaire
- Chaque [SchematiqueName] doit avoir son propre paths dans le compilerOptions dans tsconfig.json, format : "@Schema-[SchematiqueName]/*": ["src/[SchematiqueName]/*"]
- Le code doit respecter les convention KYD, c'est à dire : KISS, YAGNI & DRY
- Ainsi qu'utiliser des design patternes et les principes solid !

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/protypangel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
