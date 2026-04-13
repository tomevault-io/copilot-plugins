## pao

> This document provides a comprehensive overview of the Pao project, a real-life social deduction game. It details the project's purpose, architecture, and updated setup instructions for both the backend and mobile components.

# GEMINI.md - Pao Project

This document provides a comprehensive overview of the Pao project, a real-life social deduction game. It details the project's purpose, architecture, and updated setup instructions for both the backend and mobile components.

## Project Overview

Pao is a mobile game where players try to "eliminate" each other by taking a selfie with their assigned target in the background. The game is designed for social gatherings and is inspired by games like "Assassin" and "Among Us," but played in the real world.

### Core Gameplay

1.  **Lobby Creation:** A player hosts a game, creating a lobby with a unique code.
2.  **Joining a Game:** Other players join the lobby using the game code.
3.  **Game Start:** The host starts the game, and the backend assigns each player a secret target.
4.  **Elimination:** To eliminate a target, a player must take a photo of themselves with their target visible in the background.
5.  **Target Inheritance:** When a player successfully eliminates their target, they inherit the target of the eliminated player.
6.  **Winning:** The last player remaining is the winner.

## Tech Stack

The project is a monorepo composed of a backend service and a mobile application.

### Backend

*   **Runtime:** [Deno](https://deno.land/)
*   **Web Framework:** [Hono](https://hono.dev/)
*   **Language:** [TypeScript](https://www.typescriptlang.org/)
*   **Database:** [PostgreSQL](https://www.postgresql.org/)
*   **ORM:** [Drizzle ORM](https://orm.drizzle.team/)
*   **Real-time Communication:** [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) and [Redis](https://redis.io/)
*   **Image Storage:** S3-Compatible Object Storage (e.g., [AWS S3](https://aws.amazon.com/s3/), [Cloudflare R2](https://www.cloudflare.com/developer-platform/r2/))
*   **Containerization:** [Docker](https://www.docker.com/)

### Mobile

*   **Framework:** [React Native](https://reactnative.dev/) with [Expo](https://expo.dev/)
*   **Language:** [TypeScript](https://www.typescriptlang.org/)
*   **Styling:** [Tailwind CSS](https://tailwindcss.com/) (via NativeWind)
*   **State Management:** [Zustand](https://zustand-demo.pmnd.rs/)
*   **Navigation:** [Expo Router](https://docs.expo.dev/router/introduction/)

## Project Structure

The project is divided into two main directories:

*   `backend/`: Contains the Deno/Hono server application.
*   `mobile/`: Contains the React Native (Expo) mobile application.

```
/
├── backend/
│   ├── main.ts           # Main application entry point
│   ├── db/
│   │   └── schema.ts     # Database schema definitions
│   ├── services/
│   │   ├── auth.ts       # Authentication service
│   │   └── game/
│   │       ├── index.ts  # Game logic and API endpoints
│   │       └── ws.ts     # WebSocket handling
│   └── Dockerfile        # Docker configuration for deployment
└── mobile/
    ├── App.tsx           # Main application component (though expo-router is used)
    ├── app/              # Expo router based file-system routing
    │   ├── game.tsx      # The main game screen
    │   └── (tabs)/       # Tabbed navigation for home/create game
    ├── components/       # Reusable UI components
    ├── lib/              # Utility functions and libraries
    └── package.json      # Project dependencies and scripts
```

## Setup and Installation

### Backend Setup

1.  **Prerequisites:**
    *   [Deno](https://deno.land/manual/getting_started/installation) installed.
    *   [Docker](https://docs.docker.com/get-docker/) installed and running.

2.  **Database and Redis:**
    The backend requires a PostgreSQL database and a Redis instance. The easiest way to run these is with Docker.
    
    Create a file named `docker-compose.yml` in the `backend/` directory with the following content:
    ```yaml
    services:
      postgres:
        image: postgres:13
        environment:
          POSTGRES_USER: user
          POSTGRES_PASSWORD: password
          POSTGRES_DB: pao
        ports:
          - "5432:5432"
        volumes:
          - postgres_data:/var/lib/postgresql/data
      redis:
        image: redis:7
        ports:
          - "6379:6379"
    volumes:
      postgres_data:
    ```
    From the `backend/` directory, start the services by running:
    ```bash
    docker-compose up -d
    ```

3.  **Environment Variables:**
    Create a `.env` file in the `backend/` directory. This file holds the connection strings and credentials for the services. The backend code requires the following variables:

    ```env
    # --- Database and Redis (from docker-compose) ---
    DB_URL="postgresql://user:password@localhost:5432/pao"
    REDIS_URL="redis://localhost:6379"

    # --- Mobile App URL (for CORS) ---
    # This is not strictly needed if using a wildcard origin for development
    FRONTEND_URLS="http://localhost:8081" 

    # --- S3-Compatible Object Storage Credentials ---
    # Example using Cloudflare R2 (a free alternative to AWS S3)
    S3_BUCKET="your-bucket-name"
    S3_ACCESS_KEY="your-access-key-id"
    S3_SECRET_KEY="your-secret-access-key"
    S3_REGION="auto"
    S3_ENDPOINT="https://<your_account_id>.r2.cloudflarestorage.com"
    ```

4. **Initialize the Database:**
   With the database service running via Docker, apply the database schema. This command reads the schema from `backend/db/schema.ts` and creates the corresponding tables in the database.
   
   From the `backend/` directory, run:
   ```bash
   deno task db:push
   ```

5.  **Running the Backend:**
    Navigate to the `backend` directory and run the development server:

    ```bash
    deno task dev
    ```

    This will start the backend server on `http://localhost:8000` and watch for file changes.

    For production, use the `start` task:
    ```bash
    deno task start
    ```
    This will run the backend without watching for file changes and expects environment variables to be set in the system.

### Mobile App Setup

1.  **Prerequisites:**
    *   [Node.js](https://nodejs.org/en/download/) and [Bun](https://bun.sh/) (or npm/yarn) installed.
    *   [Expo Go](https://expo.dev/go) app on your mobile device or an Android/iOS simulator setup.

2.  **Environment Variables:**
    Create a file named `.env` in the `mobile/` directory. This tells the app where to find the backend server.

    ```env
    EXPO_PUBLIC_API_BASE="https://<your-pi-domain>/api"
    EXPO_PUBLIC_S3_BASE_URL="https://<your-r2-public-url>"
    EXPO_PUBLIC_S3_BUCKET="<your-r2-bucket-name>"
    EXPO_PUBLIC_GAME_URL="https://<your-pi-domain>/game"
    ```
    **CRITICAL:**
    *   `EXPO_PUBLIC_API_BASE` must point to your Nginx-proxied backend API endpoint, including `/api` and using `https` for production.
    *   When running the app on a physical phone for development, you **must** replace `<your-computer-ip>` with your computer's actual IP address on your local network (e.g., `192.168.1.100`). You cannot use `localhost`.

    **Important Note for `mobile/lib/axios.ts`:**
    Ensure that `mobile/lib/axios.ts` sets the `baseURL` for Axios directly to `process.env.EXPO_PUBLIC_API_BASE` without appending an extra `/api` in the code. For example:
    ```typescript
    export const api = axios.create({
      baseURL: process.env.EXPO_PUBLIC_API_BASE,
      timeout: 10000,
    });
    ```

3.  **Installation:**
    Navigate to the `mobile/` directory and install the dependencies:

    ```bash
    bun install
    ```

4.  **Running the App:**
    Start the Expo development server:

    ```bash
    bun start
    ```
    To fix potential caching issues, you can run `bun start --clear`.

    Scan the QR code that appears in your terminal with the Expo Go app to run the application.

    **Building for Web:**
    To build the static web files for deployment (e.g., on Nginx), use:
    ```bash
    bunx expo export
    ```
    The output will be in the `dist` directory.

## Database

The database schema is defined using Drizzle ORM in `backend/db/schema.ts`. To apply schema changes to your local database, run `deno task db:push` from the `backend` directory.

## Deployment

The backend is designed to be deployed as a Docker container. The provided `Dockerfile` in the `backend` directory can be used to build a container image for deployment to services like AWS ECS, Google Cloud Run, or any other container hosting platform.

## Deployment on Raspberry Pi

This section outlines how to deploy the Pao project (both the web frontend and the Deno backend) on a Raspberry Pi, using Nginx as a web server and `systemd` to manage the backend process.

### Prerequisites on Raspberry Pi

Ensure your Raspberry Pi has the following installed:

*   **Node.js (v20 or higher):**
    ```bash
    sudo apt update
    sudo apt install -y ca-certificates curl gnupg
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/nodesource.gpg
    NODE_MAJOR=20 # Or your desired Node.js version
    echo "deb [signed-by=/usr/share/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
    sudo apt update
    sudo apt install -y nodejs
    ```
*   **Bun:**
    ```bash
    curl -fsSL https://bun.sh/install | bash
    # Add Bun to PATH (add to ~/.bashrc or ~/.zshrc)
    export BUN_INSTALL="$HOME/.bun"
    export PATH="$BUN_INSTALL/bin:$PATH"
    source ~/.bashrc # Or ~/.zshrc
    ```
*   **Deno:**
    ```bash
    sudo apt install -y unzip
    curl -fsSL https://deno.land/install.sh | sh
    # Add Deno to PATH (add to ~/.bashrc or ~/.zshrc)
    export DENO_INSTALL="$HOME/.deno"
    export PATH="$DENO_INSTALL/bin:$PATH"
    source ~/.bashrc # Or ~/.zshrc
    ```
*   **Docker and Docker Compose:**
    ```bash
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    sudo usermod -aG docker $USER # Log out and back in after this
    sudo apt-get install -y docker-compose-plugin
    ```
*   **Nginx:**
    ```bash
    sudo apt update
    sudo apt install nginx -y
    ```

### Deployment Steps

**1. Clone the Project**
Clone your project repository onto your Raspberry Pi:
```bash
git clone <your-repo-url> /home/pi/pao # Adjust path as needed
cd /home/pi/pao
```

**2. Backend Setup (Database & Redis)**
Navigate to the `backend/` directory and start your database and Redis containers. Ensure your `backend/.env` file is correctly configured.
```bash
cd /home/pi/pao/backend
docker compose up -d
deno task db:push
```

**3. Build the Web Frontend**
On your Raspberry Pi (or development machine, then copy), navigate to the `mobile/` directory. Ensure your `mobile/.env` file is configured with the correct `EXPO_PUBLIC_API_BASE`, `EXPO_PUBLIC_S3_BASE_URL`, `EXPO_PUBLIC_S3_BUCKET`, and `EXPO_PUBLIC_GAME_URL` pointing to your Raspberry Pi's IP/domain.
```bash
cd /home/pi/pao/mobile
bun install
bunx expo export # Builds to the 'dist' directory
```
Copy the contents of the `dist` directory to your Nginx web root (e.g., `/var/www/html/pao-web`):
```bash
sudo mkdir -p /var/www/html/pao-web
sudo cp -r dist/* /var/www/html/pao-web/
```

**4. Configure Nginx**
Create an Nginx configuration file for your web app (e.g., `/etc/nginx/sites-available/pao-web`):
```bash
sudo nano /etc/nginx/sites-available/pao-web
```
Paste the following content, replacing `your_pi_ip_address_or_domain` with your Raspberry Pi's actual IP address or domain name:
```nginx
server {
    listen 80;
    server_name your_pi_ip_address_or_domain;

    root /var/www/html/pao-web;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://localhost:8000; # Proxy to your Deno backend
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Enable the site and restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/pao-web /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

**5. Run the Deno Backend with `systemd`**
This ensures your backend runs reliably in the background.

*   **Create a Start Script:**
    ```bash
    nano /home/pi/pao/backend/start_backend.sh
    ```
    Paste:
    ```bash
    #!/bin/bash
    # Navigate to the backend directory
    cd /home/pi/pao/backend || { echo "Failed to change directory to /home/pi/pao/backend"; exit 1; }

    # Source the .env file to load environment variables
    if [ -f ./.env ]; then
        set -a # Automatically export all variables
        . ./.env
        set +a
    else
        echo "Error: ./.env file not found! Backend will not start."
        exit 1
    fi

    # Execute the Deno backend
    /home/pi/.deno/bin/deno task start
    ```
    Make it executable:
    ```bash
sudo chmod +x /home/pi/pao/backend/start_backend.sh
    ```
*   **Create `systemd` Service File:**
    ```bash
sudo nano /etc/systemd/system/pao-backend.service
    ```
    Paste:
    ```ini
    [Unit]
    Description=Pao Game Backend
    After=network.target

    [Service]
    # Run the start script as the 'pi' user.
    # We use sudo -u pi here instead of systemd's User= directive
    # to avoid issues with environment and PATH for Deno.
    WorkingDirectory=/home/pi/pao/backend
    ExecStart=/usr/bin/sudo -u pi /home/pi/pao/backend/start_backend.sh
    Restart=always
    RestartSec=5
    StandardOutput=journal
    StandardError=journal
    # EnvironmentFile=/home/pi/pao/backend/.env # Removed: .env is now sourced in start_backend.sh

    [Install]
    WantedBy=multi-user.target
    ```
*   **Reload `systemd`, Enable, and Start:**
    ```bash
sudo systemctl daemon-reload
sudo systemctl enable pao-backend.service
sudo systemctl start pao-backend.service
    ```
*   **Check Status:**
    ```bash
sudo systemctl status pao-backend.service
journalctl -u pao-backend.service -f
    ```

### Building Android APK

To build an Android APK, use Expo Application Services (EAS) build servers.

1.  **Install EAS CLI:**
    ```bash
    npm install -g eas-cli
    ```
2.  **Log in to Expo:**
    ```bash
    eas login
    ```
3.  **Configure `eas.json`:** Ensure your `mobile/eas.json` has a profile for APK builds (e.g., `android_apk`):
    ```json
    {
      "build": {
        "android_apk": {
          "android": {
            "buildType": "apk"
          }
        },
        "production": {}
      }
    }
    ```
4.  **Build the APK:**
    Navigate to `mobile/` and run:
    ```bash
    eas build --platform android --profile android_apk
    ```
    You'll get a link to download the APK once the build is complete.ting platform.ovided `Dockerfile` in the `backend` directory can be used to build a container image for deployment to services like AWS ECS, Google Cloud Run, or any other container hosting platform.ting platform.

### Deployment Troubleshooting

This section covers common issues encountered during deployment and their solutions.

**1. Changes not reflecting after deployment (Web Frontend)**
If your web frontend changes are not visible after rebuilding and copying files:
*   **Ensure a clean build:** Always use `bunx expo export --clear` to ensure a fresh build.
*   **Correct copy command:** Verify you are copying the `dist` contents to the correct Nginx `root` directory (e.g., `sudo cp -r dist/* /var/www/your-domain/html/`).
*   **Clear browser cache:** Perform a hard refresh or clear your browser's cache for your domain.

**2. `systemd` service fails to start (`status=217/USER`, `Failed to load environment files`, `more than one ExecStart`)**
These errors often indicate issues with the `pao-backend.service` file or its interaction with the `start_backend.sh` script.
*   **Verify `start_backend.sh`:** Ensure it has the correct shebang (`#!/bin/bash`), `cd` to the backend directory, sources the `.env` file, and uses the absolute path for `deno` (e.g., `/home/pi/.deno/bin/deno task start`).
*   **Verify `pao-backend.service`:**
    *   **Remove `EnvironmentFile`:** If you're sourcing `.env` in `start_backend.sh`, remove the `EnvironmentFile=/path/to/.env` line from the service file.
    *   **Use `sudo -u pi` in `ExecStart`:** To avoid `status=217/USER` errors, use `ExecStart=/usr/bin/sudo -u pi /home/pi/pao/backend/start_backend.sh` and remove `User=` and `Group=` directives from the `[Service]` section.
    *   **Single `ExecStart`:** Ensure there is *only one* `ExecStart` line in the `[Service]` section.
*   **Reload and Restart:** Always run `sudo systemctl daemon-reload` and `sudo systemctl restart pao-backend.service` after modifying the service file.
*   **Check Logs:** Use `journalctl -u pao-backend.service -f` for detailed error messages.

**3. Camera Permissions not working on Mobile Web**
If the "Grant Permission" button does nothing or no prompt appears:
*   **Browser Console:** Check the browser's developer console for any JavaScript errors or warnings when the button is pressed.
*   **Browser Settings:** Manually check the site settings in the mobile browser to see if camera permissions are already denied or pending.
*   **Add Logging:** Temporarily add `console.log` statements to the `handleRequestPermission` function in `mobile/components/screens/ActiveGame.tsx` to trace the execution flow and permission results.
*   **Secure Context:** Ensure your site is served over `https`. Camera access often requires a secure context.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AleksanderGPL) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
