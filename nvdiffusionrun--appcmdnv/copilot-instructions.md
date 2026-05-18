## appcmdnv

> This project is a web-based application for creating and managing customer orders for a company named "NV Diffusion". It's designed to be used by a salesperson to select a client, add articles to a shopping cart, and then generate an email with the order summary.

# Project Overview

This project is a web-based application for creating and managing customer orders for a company named "NV Diffusion". It's designed to be used by a salesperson to select a client, add articles to a shopping cart, and then generate an email with the order summary.

The application is built using plain HTML, CSS, and JavaScript, with no external libraries or frameworks. It loads data from three CSV files: `AnnuaireClients.csv`, `BaseArticleTarifs.csv`, and `StockRestant.csv`.

## Key Features

*   **Client Selection:** Filter clients by sector and search by name or code.
*   **Article Catalog:**
    *   Filter articles by family and **supplier ("Fournisseur")**.
    *   Search by code or designation.
    *   Set a **default quantity** for all articles in the list.
    *   Filter controls are **sticky** on both desktop and mobile views for easy access.
*   **Dynamic Pricing:**
    *   Article prices are adjusted based on the selected client's tariff category.
    *   Correctly handles special pricing for the "ROBIN" category.
*   **Shopping Cart:**
    *   Add, remove, and update article quantities.
    *   **Visual feedback animation** when an item is added to the cart.
    *   Displays total items in the cart (badge in header).
    *   Cart section is `sticky` for constant visibility on desktop.
    *   Visually distinguishes backordered items.
*   **Stock Management:** The application displays the stock status of each article and allows ordering of out-of-stock items.
*   **Delivery Date Selection:** Users can select a desired delivery date using increment/decrement buttons, with logic to skip weekends and prevent selecting past dates.
*   **Email Generation:** On checkout, the application generates a pre-filled email with the order summary.
    *   Customized subject line format.
    *   Enhanced email body including article count, total TTC, and selected delivery date.
    *   Increased character limit for article descriptions to prevent truncation.
*   **User Interface Enhancements:**
    *   **Compact Header:** The main header has been condensed for better space usage on small screens.
    *   **Mobile-First Layout:**
        *   The floating cart button is positioned on the **bottom-left** to avoid overlapping with other controls.
        *   Article filter controls are grouped horizontally to save vertical space.
    *   Client search: Pressing 'Enter' automatically selects the first suggestion.

# Building and Running

This is a static web project. There is no build process. To run the application, you need to serve the files using a local web server.

**To start a local web server using Python:**

```bash
python3 -m http.server
```

Then, open your web browser and navigate to `http://localhost:8000`.

You can also use a tool like **ngrok** to create a temporary public URL for your local server, which is useful for testing on mobile devices that are not on the same Wi-Fi network.

**Important:**

The application expects the CSV data files to be located in a directory named `BaseAppCmd`. You should ensure this directory exists in the root of the project and contains `AnnuaireClients.csv`, `BaseArticleTarifs.csv`, and `StockRestant.csv`.

For the supplier filter to work, `BaseArticleTarifs.csv` must contain a "Fournisseur" column.

# Development Conventions

*   **Code Style:** The JavaScript code is written in a procedural style. It uses global variables to store the application's state.
*   **File Naming:** The files are named using lowercase with underscores or camelCase.
*   **CSS:** The CSS is well-structured and uses comments to separate different sections of the stylesheet.
*   **Data:** The application relies on CSV files for its data. The data parsing logic is in the `script.js` file.
*   **Dependencies:** There are no external dependencies.

# Journal des modifications

## Améliorations de linitialisation et du cœur de lapplication

*   Résolution des problèmes critiques au démarrage de lapplication (chemins de chargement CSV incorrects, erreurs de référence).
*   Refactorisation de la structure JavaScript, passant dun `script.js` unique à une architecture modulaire sous `src/`.
*   Amélioration de la robustesse du parsing des fichiers CSV pour divers formats et fins de ligne.
*   Ajout de messages derreur plus clairs pour guider lutilisateur lors des échecs de démarrage.

## Sélection dynamique des nuances et intégration au panier

*   Implémentation dun nouveau filtre "Couleurs" pour afficher une grille de nuances de produits.
*   Développement dun système de mapping basé sur les données pour associer les noms des nuances à des codes articles uniques.
*   Automatisation de la création/mise à jour des fichiers `Nuanciers/*.csv` (pour MODA, GTC, GTD, ICARE, ILIGHT, Solfine) pour intégrer les codes articles directement.
*   Ajout direct et suppression darticles au/du panier via le clic sur un bouton de nuance.
*   Ajout dun retour visuel (coche verte) sur les boutons de nuances sélectionnés.
*   Correction dun bug où le changement de gamme de nuances empêchait la sélection au premier clic.

## Améliorations UI/UX

*   Ajout de boutons `+` et `-` pour faciliter lajustement de la quantité par défaut.
*   Déplacement du champ de quantité par défaut à une position plus visible.
*   Implémentation dun bouton daction flottant (FAB) déplaçable pour le panier mobile, avec persistance de la position.

## Modale dinformations client

*   Introduction dun bouton "Information" à côté de laffichage du client sélectionné.
*   Implémentation dune fenêtre modale en lecture seule affichant les détails complets du client (Nom, Adresse, Téléphone, E-mail, etc.).
*   Ajout de boutons "Copier ladresse" et "Ouvrir dans Google Maps" au sein de la modale pour une interaction rapide.

## Corrections de bugs généraux

*   Correction dune `ReferenceError` empêchant la génération de-mails à la validation de la commande.
*   Assuré la réinitialisation complète des filtres et de linterface utilisateur du panier après la finalisation de la commande.
*   Résolution de divers bugs et incohérences mineures.

---
> Source: [nvdiffusionrun/AppCmdNV](https://github.com/nvdiffusionrun/AppCmdNV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
