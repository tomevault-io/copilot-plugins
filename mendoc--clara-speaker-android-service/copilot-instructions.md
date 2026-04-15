## clara-speaker-android-service

> L'application reçoit des textes courts (des "résumés") via des notifications push (Firebase Cloud Messaging), les transforme en audio (Text-to-Speech via l'API Google), et les lit à l'utilisateur, avec une logique spécifique liée à la connexion d'un casque Bluetooth.

### Objectif Principal

L'application reçoit des textes courts (des "résumés") via des notifications push (Firebase Cloud Messaging), les transforme en audio (Text-to-Speech via l'API Google), et les lit à l'utilisateur, avec une logique spécifique liée à la connexion d'un casque Bluetooth.

### Architecture et Composants Clés

1.  **Réception des résumés (`MyFirebaseMessagingService`)**
    *   Un message push contenant un `summaryText` arrive.
    *   Le texte est **immédiatement sauvegardé** dans la base de données locale (Room) avec le statut `isPlayed = false`.
    *   L'application vérifie si un casque Bluetooth est connecté.
        *   **Si oui :** Elle lance immédiatement la synthèse vocale et la lecture.
        *   **Si non :** Elle ne fait rien de plus. Le résumé reste en attente dans la base de données.

2.  **Base de Données (`AppDatabase`, `Summary`, `SummaryDao`)**
    *   Utilise **Room** pour stocker les résumés.
    *   Chaque `Summary` contient le texte, un booléen `isPlayed`, la date de création, le modèle de voix utilisé pour la synthèse, et le chemin vers le fichier audio une fois généré.
    *   Le `SummaryDao` fournit des méthodes pour insérer, mettre à jour, et récupérer les résumés (notamment ceux non lus).

3.  **Synthèse Vocale et Lecture (`AudioPlayerManager`, `TtsApiService`, `RetrofitClient`)**
    *   `AudioPlayerManager` est un objet singleton qui centralise toute la logique audio.
    *   Il utilise **Retrofit** (`RetrofitClient`, `TtsApiService`) pour appeler l'API Text-to-Speech de Google Cloud.
    *   Une voix française est choisie au hasard parmi une liste prédéfinie.
    *   La réponse de l'API (un contenu audio en Base64) est décodée et sauvegardée dans un fichier `.mp3` dans le stockage interne de l'application.
    *   Le chemin du fichier est alors sauvegardé dans l'entrée `Summary` correspondante dans la base de données, et le statut `isPlayed` est mis à `true`.
    *   La lecture se fait via la classe `MediaPlayer` standard d'Android.

4.  **Lecture différée via Bluetooth (`BluetoothConnectionReceiver`)**
    *   C'est une fonctionnalité clé : un `BroadcastReceiver` écoute l'événement de connexion d'un appareil Bluetooth (`ACTION_ACL_CONNECTED`).
    *   Quand un appareil se connecte, il vérifie s'il y a des résumés non lus (`isPlayed = false`) dans la base de données.
    *   **Si oui :** Il lit une petite introduction ("Vous avez X résumés en attente"), puis lance la lecture séquentielle de tous les résumés en attente, avec un délai entre chaque.

5.  **Interface Utilisateur (`MainActivity`, `SummaryViewModel`, `SummaryAdapter`)**
    *   L'interface principale (`MainActivity`) affiche une liste des **3 derniers résumés déjà joués**.
    *   Elle utilise une architecture moderne avec `ViewModel` et `LiveData` pour observer les changements dans la base de données et mettre à jour l'UI automatiquement.
    *   L'utilisateur peut cliquer sur un résumé dans la liste pour le réécouter.
    *   Au démarrage, l'activité demande la permission `BLUETOOTH_CONNECT` (nécessaire pour Android 12+) et récupère le token FCM pour les notifications.

### Résumé du Flux de Travail

1.  Un serveur externe envoie un résumé via une notification push.
2.  L'application reçoit la notification et sauvegarde le résumé.
3.  Si un casque Bluetooth est déjà connecté, le résumé est lu immédiatement.
4.  Sinon, il attend.
5.  Plus tard, lorsque l'utilisateur connecte son casque Bluetooth, l'application détecte l'événement et lit tous les résumés qui étaient en attente.
6.  L'écran principal ne sert qu'à consulter et réécouter les résumés récents.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mendoc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mendoc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
