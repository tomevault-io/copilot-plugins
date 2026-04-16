## clientwsscolarite

> **ClientScolariteWS** is a Java Web Application designed for managing student schooling information, specifically focusing on grades ("Notes"). It appears to be a client-side web interface intended to interact with a backend service or database to retrieve and display student grade information.

# ClientScolariteWS

## Project Overview
**ClientScolariteWS** is a Java Web Application designed for managing student schooling information, specifically focusing on grades ("Notes"). It appears to be a client-side web interface intended to interact with a backend service or database to retrieve and display student grade information.

## Technology Stack
- **Language:** Java 25
- **Build System:** Maven
- **Web Framework:** Jakarta EE (Servlet 6.1, JSP)
- **Dependencies:** Jersey (JAX-RS), JUnit 5

## Project Structure
The project follows the standard Maven directory structure:

- **`src/main/java/`**: Contains the Java source code.
    - `org.sid.clientscolaritews.controleur`: Contains servlets (e.g., `ControleurServlet`).
    - `org.sid.clientscolaritews.forms.beans`: Contains data models/beans (`User`, `NoteForm`).
    - `org.sid.clientscolaritews`: Contains the `HelloServlet`.
- **`src/main/webapp/`**: Contains the web resources.
    - `WEB-INF/web.xml`: Web application configuration (currently minimal).
    - `*.jsp`: JavaServer Pages for the presentation layer (`affiche.jsp`, `load.jsp`, `Notes.jsp`).
    - `*.html`: Static HTML pages (`login.html`).

## Key Files
- **`pom.xml`**: Maven configuration file defining project metadata, dependencies (Jakarta Servlet, Jersey), and build plugins.
- **`ControleurServlet.java`**: The main controller servlet intended to handle requests for student grades. It attempts to use a `Scolarite` class to fetch data and populates a `NoteForm`.
- **`login.html`**: The entry point for user authentication.
- **`load.jsp`**: Handles the processing of login data from `login.html`, creates a `User` session object, and redirects to `affiche.jsp`.

## Development Status & Known Issues
The project currently appears to be in a state of refactoring or partial implementation.

1.  **Package Mismatches**:
    - Source files are located in `org.sid.clientscolaritews.forms.beans`, but some code (e.g., `ControleurServlet`, `load.jsp`) attempts to import them from `forms.beans`. This will cause compilation errors.
2.  **Missing Classes**:
    - `ControleurServlet` references a `Scolarite` class (likely a service or logic handler) which is not present in the codebase.
3.  **Configuration**:
    - `ControleurServlet` lacks the `@WebServlet` annotation and is not defined in `web.xml`, meaning it is currently not accessible via a URL.
    - `web.xml` is effectively empty.

## Building and Running
To build the project, use the standard Maven wrapper commands. **Note:** The build is expected to fail due to the issues listed above.

```bash
# Build the project (will fail until issues are resolved)
./mvnw clean package
```

## Setup Instructions
To make the project functional, the following steps are likely required:
1.  **Fix Imports**: Update imports in Java files and JSPs to match the actual package structure (`org.sid.clientscolaritews...`).
2.  **Implement Logic**: Create or import the missing `Scolarite` class.
3.  **Configure Servlet**: Add `@WebServlet` annotation to `ControleurServlet` or configure it in `web.xml`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imranebh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
