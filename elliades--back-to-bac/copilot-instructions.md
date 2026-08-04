## back-to-bac

> Project: french-bac-game

# Project metadata
Project: french-bac-game
Description: A web app where players for each round get a random letter A–Z and must fill in words for categories (Prénom, Ville, Animal, Objet, Métier, Nourriture) within a defined time by default 3 min.
1 points per word found. 1 bonus if the word is unique. 3 points bonus if all the words are found. 5 points if all the words are unique.
There is Y rounds by default 10.
at the end of each round, the player can see the list of words he has found and the total score.
At the end of the game, it's display the scores and players ranking.


# Tech stack
– Frontend framework: React (Create React App)  
– Language: JavaScript (ES modules, functional components)  
– Styling: Tailwind CSS  
– Backend: Firebase Functions (Node.js)  
– Database: Firestore  
– Hosting: Firebase Hosting + Cloud Run via Docker  

# Docker configuration
– Base image: node:18-alpine  
– Build both frontend and functions in one stage  
– Expose build artifacts to Firebase deploy  

# CI/CD
– Use GitHub Actions to build Docker image and push to Google Container Registry  
– Deploy hosting & functions with `firebase deploy --only hosting,functions`  

# Game rules
– Categories: Prénom, Ville, Animal, Objet, Métier, Nourriture  
– Round duration: 60 seconds  
– Validate answers: +1 point each correct word starting with the target letter  
– 1 points per word found. 1 bonus if the word is unique. 3 points bonus if all the words are found. 5 points if all the words are unique.

# NPM scripts
– `start`: `react-scripts start`  
– `build`: `react-scripts build && firebase functions:build`  
– `dockerBuild`: `docker build -t french-bac-game .`  
– `deploy`: `firebase deploy --only hosting,functions`  

# Coding conventions
– Always use functional React components  
– Favor arrow functions and modern JS features  
– Keep components small and modular  
– Preserve existing comments; add JSDoc for new modules  

---
> Source: [Elliades/back-to-bac](https://github.com/Elliades/back-to-bac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
