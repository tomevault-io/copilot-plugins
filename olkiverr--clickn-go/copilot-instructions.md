## clickn-go

> ClickN'Go is a web-based e-commerce platform designed for selling PC parts, Windows keys, and computer presets. It also features a second-hand marketplace where logged-in users can sell their own parts with a commission. The project focuses on core functionalities like user accounts, a shopping cart, product management, and an admin panel. All transactions and shipments are simulated.

# ClickN'Go E-commerce Platform

## Project Overview
ClickN'Go is a web-based e-commerce platform designed for selling PC parts, Windows keys, and computer presets. It also features a second-hand marketplace where logged-in users can sell their own parts with a commission. The project focuses on core functionalities like user accounts, a shopping cart, product management, and an admin panel. All transactions and shipments are simulated.

## Key Features Implemented

*   **Real-time Chat System:** For communication between buyers and sellers on marketplace items.
*   **Admin Panel:** Comprehensive management for users, products, orders, and promotions.
*   **Second-Hand Marketplace:** CRUD operations for marketplace items, with user ownership enforcement.
*   **Coupon Code/Promo System:** Apply promo codes to carts, with discount calculations and order integration.
*   **Multi-Step Checkout Process:** Includes a review step before order finalization.
*   **User Account Page:** View profile information, order history, and edit account details.
*   **Database Initialization:** Robust script for creating all necessary tables and populating sample data.

## Technologies Used
*   **Backend:** Node.js, Express.js
*   **Database:** MariaDB (via `mysql2/promise`)
*   **Templating:** EJS
*   **Authentication:** `bcrypt` for password hashing, `express-session` for session management.
*   **Real-time Communication:** `socket.io`
*   **Frontend:** HTML5, CSS3, JavaScript
*   **Containerization:** Docker, Docker Compose

## Building and Running

### Prerequisites
*   Node.js (LTS version recommended)
*   npm (Node Package Manager)
*   MariaDB or MySQL server running locally (e.g., via WAMP, XAMPP, MAMP, or Docker).
*   Docker (if using Docker for deployment)

### 1. Install Dependencies
Navigate to the project root directory and install the required Node.js packages:
```bash
npm install
```

### 2. Database Setup
Initialize your database schema and populate it with sample data and an admin user:
```bash
npm run db:init
```
*   **Admin User:** An admin user will be created with `username: admin`, `email: admin@gmail.com`, and `password: admin`. You will be prompted to change these credentials on your first login.

### 3. Run the Application

#### Using Node.js (Local Development)
Start the Node.js server:
```bash
npm start
```

#### Using Docker (Recommended for Production/Consistent Environments)

1.  **Ensure Docker is Running:** Make sure Docker Desktop or your Docker daemon is active.

2.  **Build and Run Containers:** Navigate to the project root directory and execute:
    ```bash
    docker-compose up --build
    ```

3.  **Initialize Database (for Docker):**
    After the containers are up and running for the first time, you need to initialize the database within the Docker environment. Open a new terminal in the project root and run:
    ```bash
    docker-compose exec app npm run db:init
    ```

### 4. Access the Application
Open your web browser and navigate to:
```
http://localhost:3000
```

## Development Conventions
*   **Code Structure:** Routes are organized in `src/routes`, middleware in `src/middleware`, and views in the `views` directory.
*   **Styling:** Uses `public/css/main.css` and `public/css/theme.css`.
*   **Database Interaction:** Uses a connection pool from `src/db.js`.
*   **Authentication:** Session-based, with `requireLogin` and `requireAdmin` middleware.
*   **Environment Variables:** Uses `.env` file for database configuration.

## Next Steps
All planned features are complete. Further development could include:
*   PWA (Progressive Web App) implementation.
*   Client Service integration.
*   Automatic product data integration (e.g., from `https://github.com/docyx/pc-part-dataset`).
*   Further UI/UX refinements and testing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/olkiverr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/olkiverr)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
