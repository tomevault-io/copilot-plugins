## git-workflow

> Commits et pushes Git après changements importants (Velour)


# Workflow Git (agent + contributeurs)

Après un **lot cohérent** de modifications (feature, correctif sensible, refacto, durcissement prod, mise à jour deps) :

1. Vérifier le dépôt : `git status`
2. Après toute modification de fichiers **`.dart`** : **`dart format .`** avant commit (la CI GitHub « Flutter CI » exécute `dart format --set-exit-if-changed .` ; sans formatage, le job échoue vite).
3. Lancer si pertinent : `dart analyze`, `flutter test` (ou au minimum après toucher `lib/` ou `test/`)
4. **Commit** : message clair, style conventionnel si possible (`feat:`, `fix:`, `refactor:`, `docs:`, `chore:`)
5. **Push** vers `origin` dès que le lot est stable (pas de demi-travail laissé des jours sans commit)

Préférer **un commit = une intention** (facilite review et `git bisect`). Éviter un énorme diff qui mélange formatage global et logique métier quand c’est évitable.

Ne pas demander à l’utilisateur de « faire le git » à la place de l’agent si l’environnement permet `git_write` / réseau : exécuter `git add`, `git commit`, et `git push` quand c’est demandé ou à la fin d’une mission importante.

## Builds store (Android AAB / iOS IPA)

Avant **chaque** nouvelle livraison Play ou App Store / TestFlight : **incrémenter** dans `pubspec.yaml` le numéro après `+` (ex. `1.0.0+7` → `+8`). Ce chiffre est à la fois le **versionCode** Android et le **CFBundleVersion** iOS — il doit **toujours monter** ; sinon Play refuse l’upload. Après changement : `flutter pub get`, puis builds ; **commit** le bump de version avec le lot ou juste avant le push des artefacts.

**Notes « Nouveautés » / Release notes (fiches dans le dépôt)** : à chaque lot **store-ready** qui prépare une livraison (bump `pubspec`, builds AAB/IPA, ou validation explicite « prêt store »), **remplacer** dans `docs/store_listings/` (`en.md`, `fr.md`, `de.md`, `zh-Hans.md`, `hi.md`) le texte des sections **Nouveautés de cette version** (App Store) et **Notes de version** (Play) par les puces **réelles** du binaire ; aligner la **référence de version** citée dans ces blocs sur `pubspec.yaml` (`version:` / `+N`). Ne pas conserver d’anciennes notes comme si elles décrivaient la nouvelle release.

**Convention Velour (version marketing + build)** : à chaque build store, monter **à la fois** la partie avant `+` et le `+N` — enchaîner les sorties mineures **1.0.0 → 1.1.0 → 1.2.0** (trois segments) **et** `+16 → +17 → +18` (ex. `1.1.0+17`, puis `1.2.0+18`). Ne jamais réutiliser un `+N` déjà uploadé sur un store.

**Dart-defines obligatoires** pour un binaire **store** aligné prod Velour (ne pas oublier quand l’utilisateur demande les builds) :

1. `--dart-define=VELOUR_PRIVACY_POLICY_URL=<URL HTTPS réelle>` (même URL que la fiche store / Notion).
2. `--dart-define=VELOUR_ANALYTICS=true` — active Firebase Analytics + clé Crashlytics `velour_analytics` ; la **politique de confidentialité** doit décrire cette collecte. Désactiver uniquement si demande explicite : `VELOUR_ANALYTICS=0 ./scripts/build_store_artifacts.sh`.

Référence : `./scripts/build_store_artifacts.sh` (AAB + IPA), `./scripts/echo_store_release_build.sh`, `docs/STORE_RELEASE_CHECKLIST.md` §1.

### Avant de dire à l’utilisateur que les builds sont « store ready »

**Toujours exécuter** (ou confirmer que la CI équivalente vient de passer au vert sur le même `main`) :

1. `dart format --set-exit-if-changed .`
2. `dart analyze --fatal-infos`
3. `flutter test`
4. `flutter build web --release --no-wasm-dry-run -O2`
5. `cd functions && npm ci && npm run build`

**Si le lot touche l’audio** (`lib/services/audio_handler.dart`, session, bootstrap audio) : relire le flux **verrou natif / préload / one-shots** ; ne pas supposer que la CI entend le timing.

**Si le lot touche les icônes iOS** (`ios/Runner/Assets.xcassets/AppIcon.appiconset`) : vérifier que les variantes **Dark** (et au besoin **Tinted**) restent **lisibles** sur fond Springboard (iOS 18 : thèmes d’icônes sombre / teinté) — idéalement aperçu Xcode ou build installée ; on peut vérifier en dépôt que les PNG **Dark** sont au moins identiques aux universels si on réutilise le même visuel.

**Limite honnête** : « store ready » côté agent = **zéro dette connue dans le dépôt ci-dessus** + checklist humaine (`docs/STORE_RELEASE_CHECKLIST.md`, appareils réels, son, icône). **Ne pas** présenter cela comme un audit sensoriel complet sans au moins une **build release** sur device ou la validation explicite du contributeur.

---
> Source: [croustibat0630/velour_app](https://github.com/croustibat0630/velour_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
