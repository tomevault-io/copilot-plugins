## signature-pad-demo

> This document provides a comprehensive overview of the L9G Signature Pad Demo project for the Gemini AI assistant.

# Gemini Context: L9G Signature Pad Demo

This document provides a comprehensive overview of the L9G Signature Pad Demo project for the Gemini AI assistant.

## Project Overview

The project is a web application that demonstrates capturing and validating digital signatures. It is built with Java and Spring Boot, featuring a frontend that uses Thymeleaf, HTMX, and plain JavaScript.

The application allows a user to register a "signature pad" (a browser instance), which can then be used to sign documents. The signing process involves displaying a document, allowing the user to draw a signature, and then submitting the signature for validation. The application also uses WebSockets for real-time communication between the server and the signature pad client.

### Key Technologies

*   **Backend:** Java 21, Spring Boot, Spring Web, Spring Security, Spring WebSocket
*   **Frontend:** Thymeleaf, HTMX, JavaScript, Bootstrap, Font Awesome
*   **Build:** Apache Maven
*   **Security:** Nimbus JOSE & JWT for handling JSON Web Signatures.
*   **Frontend Dependencies:** Frontend JavaScript libraries (`jose`, `sweetalert2`, `signature_pad`) are managed via the `LOAD_REQUIRED_JS_LIBS.sh` script, which downloads them from CDNs.

## Building and Running

### 1. Download Frontend Dependencies

Before the first run, execute the following script to download the required JavaScript libraries:

```bash
./LOAD_REQUIRED_JS_LIBS.sh
```

### 2. Build the Project

Use Maven to build the project:

```bash
mvn clean install
```

### 3. Run the Application

You can run the application using the Spring Boot Maven plugin:

```bash
mvn spring-boot:run
```

Alternatively, you can run the packaged JAR file:

```bash
java -jar target/signature-pad-demo.jar
```

The application will be available at `http://localhost:8080`.

## Development Conventions

*   **Configuration:** Application settings are managed in `src/main/resources/application.yaml`. Local overrides can be placed in a `config.yaml` file in the project root.
*   **Backend Code:** All Java source code is located in `src/main/java/l9g/webapp/signaturepaddemo`.
*   **Frontend Code:**
    *   HTML templates are in `src/main/resources/templates`.
    *   CSS is in `src/main/resources/static/css`.
    *   JavaScript is in `src/main/resources/static/js`.
*   **Security:** The application uses a custom Spring Security configuration, defined in `src/main/java/l9g/webapp/signaturepaddemo/config/WebSecurityConfig.java`.
*   **Dependencies:** Backend dependencies are managed in `pom.xml`. Frontend dependencies are managed manually via the `LOAD_REQUIRED_JS_LIBS.sh` script.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thorsten-l)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thorsten-l)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
