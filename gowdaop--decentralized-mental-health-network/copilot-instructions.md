## decentralized-mental-health-network

> This document provides instructions for Gemini on how to interact with the "Decentralized Mental Health Network" project.

# Gemini Project: Decentralized Mental Health Network

This document provides instructions for Gemini on how to interact with the "Decentralized Mental Health Network" project.

## Project Overview

This is a full-stack application that provides a decentralized mental health support network. It consists of three main parts:

1.  **Frontend:** A web interface for users to interact with the platform.
2.  **Backend:** A server that handles business logic and data storage.
3.  **Blockchain:** Smart contracts that manage the decentralized aspects of the network.

## Project Structure

-   `frontend/`: Contains the frontend application code.
-   `backend/`: Contains the backend server code.
-   `blockchain/`: Contains the smart contracts and related blockchain code.
-   `docs/`: Contains project documentation.

## Development Instructions

### Frontend

-   **Technology:** React, TypeScript, Tailwind CSS
-   **Package Manager:** npm
-   **To install dependencies:** `npm install` in the `frontend` directory.
-   **To run:** `npm start` in the `frontend` directory.
-   **To build:** `npm run build` in the `frontend` directory.
-   **To test:** `npm test` in the `frontend` directory.

### Backend

-   **Technology:** Python, FastAPI
-   **Package Manager:** pip
-   **To install dependencies:** `pip install -r requirements.txt` in the `backend` directory.
-   **To run:** `uvicorn main:app --reload` in the `backend` directory.
-   **To test:** `python run_all_tests.py` in the `backend` directory.

### Blockchain

-   **Technology:** Solidity, Hardhat
-   **Package Manager:** npm
-   **To install dependencies:** `npm install` in the `blockchain` directory.
-   **To compile contracts:** `npx hardhat compile` in the `blockchain` directory.
-   **To run tests:** `npx hardhat test` in the `blockchain` directory.
-   **To deploy contracts:** `npx hardhat run scripts/deploy.js --network <network-name>` in the `blockchain` directory.

## Deployment

The `deploy.sh` script in the root directory can be used to deploy the application.

---
> Source: [gowdaop/decentralized-mental-health-network](https://github.com/gowdaop/decentralized-mental-health-network) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
