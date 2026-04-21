## offlinepaymentsystem

> - **Frontend**: User registers and logs in using their phone number and password, with OTP verification for online sessions.

# App Flow Document

## 1. User Onboarding and Authentication
- **Frontend**: User registers and logs in using their phone number and password, with OTP verification for online sessions.
- **Backend**: Standard user creation and JWT issuance upon successful authentication. The backend creates a corresponding wallet for the user in MongoDB with an initial balance of 0.
- **Frontend**: On successful login, the app securely stores the JWT for online API calls.

## 2. Wallet Loading (Online Only) 💰
- **Frontend**: From the main wallet screen, the user initiates an "Add Money" flow. They enter the amount they wish to add.
- **Backend**: The app sends an authenticated request to the backend to create a Razorpay order. The backend returns an `order_id`.
- **Frontend**: The app opens the Razorpay payment interface with the `order_id`. The user completes the payment using their preferred online method.
- **Backend**: Upon successful payment, Razorpay sends a confirmation to the backend. The backend verifies the payment signature, updates the user's `walletBalance` in the **MongoDB** database, and logs the transaction.
- **Frontend**: The app UI updates in real-time to reflect the new, higher wallet balance. This balance is then cached locally in the **SQLite** database during the next sync.

## 3. Money Transfer (Offline) 📲
- **Frontend**: When the app detects it's offline, it operates using the balance stored in the local **SQLite** database. To make a payment, the user enters the recipient's details and amount.
- **Device**: The app checks if the local balance is sufficient. If yes, it **deducts the amount from the local SQLite database immediately** and queues the transaction. It then encrypts the transaction data (sender, receiver, amount, timestamp) using AES-256.
- **SMS Relay**: The app prompts for SMS permissions and sends the encrypted payload via SMS to the Tasker relay service.
- **Backend**: The Tasker relay forwards the data to the backend endpoint. The backend decrypts the payload, validates the transaction against its **MongoDB** records, updates the respective user wallets, and logs the transaction.
- **SMS Confirmation**: The backend sends an encrypted confirmation SMS back to the user's device via Twilio.
- **Frontend**: A background listener reads and decrypts the confirmation SMS. It updates the status of the transaction in the local SQLite database from "Queued" to "Completed." The UI reflects this change.

## 4. Robust Data Synchronization 🔄
- **Detection**: The app continuously monitors network status. When connectivity is restored, it automatically triggers the sync process.
- **Two-Way Sync**:
    1.  **Frontend → Backend**: The app sends all "Queued" or "Completed" offline transactions from its SQLite database to the backend.
    2.  **Backend → Frontend**: The backend processes these transactions, reconciles any differences, and returns the authoritative wallet balance and complete transaction history from **MongoDB**.
- **Reconciliation**: The frontend updates its local SQLite database with the data received from the backend, ensuring the local balance and history perfectly match the server's state. This resolves any potential discrepancies and prepares the app for the next offline session.

## 5. Transaction History
- **Frontend**: The app displays a unified transaction list fetched from the local SQLite database. Each transaction is clearly tagged as **"Online"** (for wallet top-ups) or **"Offline"** (for wallet-to-wallet transfers) to provide full transparency to the user.

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SarveshHase) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
