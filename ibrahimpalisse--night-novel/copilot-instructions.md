## night-novel

> **TOUJOURS utiliser les regex centralisées de `backend/utils/validation.js`**

# 🔒 RÈGLES DE VALIDATION REGEX - BACKEND

## ⚠️ RÈGLE CRITIQUE : UTILISATION DES REGEX CENTRALISÉES

**TOUJOURS utiliser les regex centralisées de `backend/utils/validation.js`**

### 📋 REGEX DISPONIBLES

Toutes les regex suivantes sont définies et exportées dans `backend/utils/validation.js` :

1. **REGEX_USERNAME** - `/^[a-zA-Z0-9_-]+$/`
   - Validation des noms d'utilisateur
   - Utilisation : `const { REGEX_USERNAME } = require('./utils/validation');`

2. **REGEX_PASSWORD** - `/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]+$/`
   - Validation des mots de passe
   - Utilisation : `const { REGEX_PASSWORD } = require('./utils/validation');`

3. **REGEX_IMAGE_EXTENSION** - `/\.(jpg|jpeg|png|gif|webp)$/i`
   - Validation des extensions d'images
   - Utilisation : `const { REGEX_IMAGE_EXTENSION } = require('./utils/validation');`

4. **REGEX_VERIFICATION_CODE** - `/^\d{6}$/`
   - Validation des codes de vérification (6 chiffres)
   - Utilisation : `const { REGEX_VERIFICATION_CODE } = require('./utils/validation');`

5. **REGEX_ALPHANUMERIC** - `/^[a-zA-Z0-9]+$/`
   - Validation des codes alphanumériques
   - Utilisation : `const { REGEX_ALPHANUMERIC } = require('./utils/validation');`

6. **REGEX_NUMERIC_ID** - `/^\d+$/`
   - Validation des IDs numériques
   - Utilisation : `const { REGEX_NUMERIC_ID } = require('./utils/validation');`

7. **REGEX_DANGEROUS_FILENAME_CHARS** - `/[<>:"|?*\x00-\x1f]/`
   - Détection des caractères dangereux dans les noms de fichiers
   - Utilisation : `const { REGEX_DANGEROUS_FILENAME_CHARS } = require('./utils/validation');`

8. **REGEX_DANGEROUS_CONTROL_CHARS** - `/[\x00-\x08\x0B\x0C\x0E-\x1F\x7F]/`
   - Détection des caractères de contrôle dangereux
   - Utilisation : `const { REGEX_DANGEROUS_CONTROL_CHARS } = require('./utils/validation');`

9. **REGEX_DANGEROUS_FOLDER_CHARS** - `/[<>"&'\/\\|{}\[\]()=+*?^$#@!%~`\x00-\x1F\x7F-\x9F]/`
   - Détection des caractères dangereux dans les noms de dossiers
   - Utilisation : `const { REGEX_DANGEROUS_FOLDER_CHARS } = require('./utils/validation');`

10. **REGEX_NAVIGATION_SEQUENCES** - `/\.\.|\.\/|\/\.|\\|\.\.\\/`
    - Détection des séquences de navigation (path traversal)
    - Utilisation : `const { REGEX_NAVIGATION_SEQUENCES } = require('./utils/validation');`

11. **REGEX_RESERVED_FILENAMES** - `/^(CON|PRN|AUX|NUL|COM[1-9]|LPT[1-9])$/i`
    - Détection des noms de fichiers réservés Windows
    - Utilisation : `const { REGEX_RESERVED_FILENAMES } = require('./utils/validation');`

12. **REGEX_LEADING_TRAILING_SPACES** - `/^\s+|\s+$/`
    - Détection des espaces en début/fin de chaîne
    - Utilisation : `const { REGEX_LEADING_TRAILING_SPACES } = require('./utils/validation');`

13. **REGEX_DANGEROUS_SEARCH_CHARS** - `/[<>\"'&]/`
    - Détection des caractères dangereux dans les recherches
    - Utilisation : `const { REGEX_DANGEROUS_SEARCH_CHARS } = require('./utils/validation');`

---

## ✅ RÈGLES À SUIVRE

### 1. **TOUJOURS importer les regex centralisées**

❌ **MAUVAIS** :
```javascript
// Ne JAMAIS faire ça
const emailCodeRegex = /^\d{6}$/;
if (emailCodeRegex.test(code)) { ... }
```

✅ **BON** :
```javascript
// TOUJOURS faire ça
const { REGEX_VERIFICATION_CODE } = require('../utils/validation');
if (REGEX_VERIFICATION_CODE.test(code)) { ... }
```

### 2. **Si une regex n'existe pas, l'ajouter à `backend/utils/validation.js`**

Avant d'utiliser une regex en dur :
1. Vérifier si elle existe déjà dans `backend/utils/validation.js`
2. Si elle n'existe pas, l'ajouter avec un nom descriptif
3. L'exporter : `module.exports.REGEX_MON_NOM = REGEX_MON_NOM;`
4. Utiliser la constante importée dans le code

### 3. **Corriger automatiquement les fichiers qui n'utilisent pas les regex centralisées**

Si tu trouves un fichier qui utilise une regex en dur correspondant à une regex centralisée :
1. Identifier la regex en dur
2. Trouver la constante correspondante dans `backend/utils/validation.js`
3. Importer la constante
4. Remplacer la regex en dur par la constante importée

### 4. **Exceptions autorisées**

Certaines regex sont spécifiques à leur contexte et ne doivent PAS être centralisées :
- Regex pour slugs (support Unicode spécifique)
- Regex pour validation email RFC 5322
- Regex pour détection IP privées (SSRF)
- Regex pour validation IPv4/IPv6
- Regex pour nonces hexadécimaux
- Regex pour sanitization SQL LIKE
- Regex pour normalisation IP IPv6 mappées
- Regex pour parsing tailles de fichiers

Ces regex doivent rester dans leurs fichiers respectifs avec un commentaire justifiant leur existence.

---

## 🔍 EXEMPLES DE CORRECTIONS

### Exemple 1 : Code de vérification

❌ **AVANT** :
```javascript
// backend/controllers/auth/authController.js
const codeRegex = /^\d{6}$/;
if (!codeRegex.test(code)) {
  return res.status(400).json({ error: 'Code invalide' });
}
```

✅ **APRÈS** :
```javascript
// backend/controllers/auth/authController.js
const { REGEX_VERIFICATION_CODE } = require('../../utils/validation');
if (!REGEX_VERIFICATION_CODE.test(code)) {
  return res.status(400).json({ error: 'Code invalide' });
}
```

### Exemple 2 : Extension d'image

❌ **AVANT** :
```javascript
// backend/middleware/UploadMiddleware.js
const imageExtRegex = /\.(jpg|jpeg|png|gif|webp)$/i;
if (!imageExtRegex.test(filename)) {
  return res.status(400).json({ error: 'Format non supporté' });
}
```

✅ **APRÈS** :
```javascript
// backend/middleware/UploadMiddleware.js
const { REGEX_IMAGE_EXTENSION } = require('../utils/validation');
if (!REGEX_IMAGE_EXTENSION.test(filename)) {
  return res.status(400).json({ error: 'Format non supporté' });
}
```

### Exemple 3 : Nom d'utilisateur

❌ **AVANT** :
```javascript
// backend/controllers/user/profileController.js
const usernameRegex = /^[a-zA-Z0-9_-]+$/;
if (!usernameRegex.test(username)) {
  return res.status(400).json({ error: 'Nom invalide' });
}
```

✅ **APRÈS** :
```javascript
// backend/controllers/user/profileController.js
const { REGEX_USERNAME } = require('../../utils/validation');
if (!REGEX_USERNAME.test(username)) {
  return res.status(400).json({ error: 'Nom invalide' });
}
```

---

## 🎯 CHECKLIST LORS DE L'ÉDITION D'UN FICHIER

Avant de modifier un fichier backend :

- [ ] Vérifier s'il contient des regex en dur
- [ ] Pour chaque regex trouvée :
  - [ ] Vérifier si elle correspond à une regex centralisée
  - [ ] Si oui, remplacer par l'import de la constante
  - [ ] Si non, vérifier si elle doit être centralisée
  - [ ] Si elle doit être centralisée, l'ajouter à `backend/utils/validation.js`

---

## 📝 NOTES IMPORTANTES

1. **Sécurité** : Toutes les regex centralisées sont optimisées pour éviter les attaques ReDoS
2. **Cohérence** : Utiliser les mêmes regex garantit un comportement uniforme
3. **Maintenance** : Modifier une regex en un seul endroit facilite la maintenance
4. **Documentation** : Les regex centralisées sont documentées dans `backend/utils/validation.js`

---

**Dernière mise à jour** : 2025-12-04  
**Fichier de référence** : `backend/utils/validation.js`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ibrahimpalisse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
