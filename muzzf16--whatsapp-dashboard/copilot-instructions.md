## whatsapp-dashboard

> This project is a WhatsApp dashboard that allows users to send messages, view the connection status, and see incoming messages. It consists of a React frontend and a Node.js backend. The backend uses the Baileys library to connect to WhatsApp.

# GEMINI.md

## Project Overview

This project is a WhatsApp dashboard that allows users to send messages, view the connection status, and see incoming messages. It consists of a React frontend and a Node.js backend. The backend uses the Baileys library to connect to WhatsApp.

**Frontend:**
- Framework: React
- Styling: Tailwind CSS
- Communication with backend: Axios for HTTP requests and Socket.IO for real-time updates.

**Backend:**
- Framework: Express.js
- WhatsApp Integration: Baileys
- Real-time communication: Socket.IO

## Features

- **Multi-device Connection:** Manage multiple WhatsApp accounts simultaneously.
- **Send Messages:** Send text messages, images, and documents (up to 25MB) to individuals and groups.
- **Broadcast Messages:** Send messages to multiple recipients at once.
- **Real-time Status:** View the real-time connection status of each account.
- **Message Logs:** View logs of incoming and outgoing messages for each account, with support for group messages and filtering by sender/recipient.
- **Webhook Integration:** Securely integrate with services like n8n using signed webhooks.

## Building and Running

### Client (Frontend)

1.  Navigate to the `client` directory:
    ```bash
    cd client
    ```
2.  Install dependencies:
    ```bash
    npm install
    ```
3.  Start the development server:
    ```bash
    npm start
    ```
    The client will be available at `http://localhost:3000`.

### Server (Backend)

1.  Navigate to the `server` directory:
    ```bash
    cd server
    ```
2.  Install dependencies:
    ```bash
    npm install
    ```
3.  Start the server:
    ```bash
    npm start
    ```
    The server will be running on `http://localhost:4000`.

## API Endpoints

The backend provides the following API endpoints:

- `POST /api/connections/start`: Starts a new WhatsApp connection.
- `POST /api/connections/disconnect`: Disconnects a WhatsApp connection.
- `POST /api/connections/disconnect-all`: Disconnects all WhatsApp connections.
- `GET /api/connections`: Gets a list of all active connections.
- `POST /api/:connectionId/send-message`: Sends a message to a specified number or group using the specified connection.
- `POST /api/:connectionId/broadcast-message`: Sends a broadcast message to multiple numbers using the specified connection.
- `GET /api/:connectionId/status`: Gets the current connection status of the specified WhatsApp client.
- `GET /api/:connectionId/messages`: Gets the log of incoming messages for the specified connection.
- `GET /api/:connectionId/outgoing-messages`: Gets the log of outgoing messages for the specified connection.
- `GET /api/:connectionId/qrcode`: Gets the QR code for logging in to WhatsApp for the specified connection.
- `GET /api/webhook`: Gets the webhook URL and secret.
- `POST /api/webhook`: Updates the webhook URL and secret.

## Webhook Integration

The application can send a POST request to a specified webhook URL whenever a new message is received. The webhook URL and a secret for signing the requests can be configured via the dashboard.

To ensure the security of your webhooks, the application signs the request payload using HMAC-SHA256 with the configured secret. The signature is sent in the `X-Webhook-Signature` header.

The webhook payload has the following structure:

```json
{
  "event": "new_message",
  "connectionId": "<connection_id>",
  "sender": "<sender_number>",
  "message": "<message_text>",
  "timestamp": "<timestamp>",
  "groupName": "<group_name>",
  "senderName": "<sender_name>",
  "originalMessage": { ... }
}
```

## Configuration

The backend can be configured via the `server/config.json` file. This file contains the webhook URL and secret.

```json
{
  "webhookUrl": "",
  "webhookSecret": ""
}
```

## Development Conventions

- The frontend uses React with functional components and hooks.
- The backend is structured into `controllers`, `routes`, and `services` directories.
- The `gemini.txt` file contains instructions for adding new features to the application. It seems to be a log of changes or a set of instructions for the developer.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muzzf16)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/muzzf16)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
