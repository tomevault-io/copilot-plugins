## padelathome

> This is a Node.js project for a paddle tennis court booking system called "Padel at Home". It's a full-stack application with a backend built using the Express framework and a frontend with static HTML, CSS, and JavaScript files. The backend provides a REST API for managing users, courts, bookings, and schedules.

# Project Overview

This is a Node.js project for a paddle tennis court booking system called "Padel at Home". It's a full-stack application with a backend built using the Express framework and a frontend with static HTML, CSS, and JavaScript files. The backend provides a REST API for managing users, courts, bookings, and schedules.

## Key Technologies

*   **Backend:** Node.js, Express.js
*   **Database:** PostgreSQL
*   **Authentication:** JSON Web Tokens (JWT)
*   **Frontend:** HTML, CSS, JavaScript

## Architecture

The project follows a typical Model-View-Controller (MVC) like pattern, with:

*   `server.js`: The main entry point of the application.
*   `src/api`: Contains the route definitions for the REST API.
*   `src/controllers`: Contains the business logic for each API route.
*   `src/config`: Contains the database configuration.
*   `public`: Contains the static frontend files.

# Building and Running

## Prerequisites

*   Node.js and npm
*   PostgreSQL

## Installation

1.  Clone the repository.
2.  Install the dependencies:
    ```bash
    npm install
    ```
3.  Create a `.env` file in the root of the project and add the following environment variables:
    ```
    PORT=3000
    DB_USER=your_db_user
    DB_HOST=your_db_host
    DB_DATABASE=your_db_name
    DB_PASSWORD=your_db_password
    DB_PORT=your_db_port
    JWT_SECRET=your_jwt_secret
    ```

## Running the Application

To start the server, run the following command:

```bash
npm start
```

The server will start on the port specified in your `.env` file (default is 3000).

## Testing

There are no tests configured for this project.

```json
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "node server.js"
}
```

# Development Conventions

*   The code is written in JavaScript.
*   The project uses a modular structure, with different functionalities separated into different files and directories.
*   The API routes are defined in the `src/api` directory, and the corresponding controllers are in the `src/controllers` directory.
*   Database queries are performed using the `pg` library.
*   Authentication is handled using JWTs.
*   Environment variables are used for configuration.

---

# Padel@Home Application Requirements

## 1.1 Propósito del Documento:
Este documento detalla los requisitos funcionales y no funcionales para la aplicación "Padel@Home". Sirve como una guía maestra para el desarrollo, describiendo el comportamiento del sistema, las interacciones del usuario y las reglas de negocio.

## 1.2 Visión del Producto:
"Padel@Home" es una Progressive Web App (PWA) diseñada para gestionar de forma eficiente y sencilla la reserva de pistas de pádel en comunidades residenciales privadas. La aplicación busca facilitar la autogestión de las reservas por parte de los residentes, fomentar la comunidad a través de partidas abiertas y maximizar el uso de las instalaciones.

## 1.3 Alcance de la Versión Actual:
Esta especificación cubre la aplicación base (V1) junto con las funcionalidades avanzadas de Lista de Espera y Partidas Abiertas (V2).

## 2.0 Perfiles de Usuario y Roles

### 2.1 Residente (Rol: user):
*   Puede tener una cuenta creada por un administrador.
*   Puede iniciar sesión, ver su perfil y restablecer su contraseña.
*   Puede visualizar un calendario semanal con la disponibilidad de las pistas.
*   Puede crear reservas privadas o proponer "Partidas Abiertas".
*   Puede unirse a Partidas Abiertas creadas por otros residentes.
*   Puede apuntarse a la Lista de Espera de horarios ya ocupados.
*   Puede cancelar sus propias reservas o abandonar las partidas a las que se ha unido.

### 2.2 Administrador de la Comunidad (Rol: admin):
*   Posee todos los permisos de un Residente.
*   Accede a un Panel de Administración para gestionar la plataforma.
*   Gestiona las cuentas de los usuarios: crea nuevas cuentas (invitaciones), aprueba registros, activa y desactiva cuentas.
*   Gestiona la infraestructura: crea y edita las pistas y las direcciones/edificios válidos de la comunidad.
*   Gestiona el calendario: puede bloquear horarios y cancelar cualquier reserva.
*   Gestiona los ajustes generales de la aplicación (horarios de apertura, reglas de reserva, etc.).

## 3.0 Requisitos Funcionales Detallados

### 3.1 Módulo de Autenticación y Cuentas

*   **FR-AUTH-01: Creación de Cuentas por Administrador:** El registro público puede ser deshabilitado. El flujo principal es que el administrador cree una cuenta para un residente introduciendo su nombre, email y dirección (seleccionando de una lista predefinida).
*   **FR-AUTH-02: Establecimiento de Contraseña por el Usuario:** Al ser creada su cuenta, el usuario recibe un email con un enlace único y de tiempo limitado para establecer su propia contraseña e iniciar sesión por primera vez.
*   **FR-AUTH-03: Inicio de Sesión Seguro:** Los usuarios acceden con email y contraseña. Las contraseñas se almacenan de forma segura (hasheadas con bcrypt).
*   **FR-AUTH-04: Gestión de Sesión con Tokens JWT:** Tras el login, el sistema genera un token JWT que se usa para autenticar las peticiones posteriores a la API. El token tiene una expiración de 24 horas.

### 3.2 Módulo de Sistema de Reservas

*   **FR-BOOK-01: Visualización de Disponibilidad:** La interfaz principal es un calendario semanal (L-D) que muestra cada franja horaria. Los slots se colorean según su estado:
    *   Verde (Libre): Disponible para reservar.
    *   Amarillo (Partida Abierta): Un usuario ha reservado el slot y lo ha abierto para que otros se unan. Muestra el número de participantes (ej. "2/4").
    *   Rojo (Ocupado): Reserva privada o Partida Abierta ya completa.
    *   Gris (Bloqueado): Bloqueado por un administrador.
*   **FR-BOOK-02: Creación de Reserva:** Al hacer clic en un slot libre, un popup permite al usuario reservar por 60 o 90 minutos (si hay tiempo suficiente).
*   **FR-BOOK-03: Cancelación/Abandono:**
    *   El dueño de una reserva puede cancelarla.
    *   Un participante de una partida abierta puede abandonarla.
*   **FR-BOOK-04: Notificaciones por Correo:** Las confirmaciones de reserva se envían por email y deben incluir un archivo adjunto .ics para añadir el evento al calendario del usuario.
*   **FR-BOOK-05: Reglas de Negocio:**
    *   Un usuario solo puede tener una reserva activa a la vez (no aplica si se une a partidas abiertas de otros).
    *   El número de días de antelación para reservar es configurable por el admin.

### 3.3 Módulo de Partidas Abiertas

*   **FR-OPEN-01: Creación:** Al reservar, el usuario puede marcar una casilla para definir la reserva como "Partida Abierta", estableciendo el número de jugadores en 4. El creador se añade como primer participante.
*   **FR-OPEN-02: Unirse:** Cualquier usuario puede hacer clic en un slot "amarillo" y unirse a la partida hasta que se llene.
*   **FR-OPEN-03: Reglas de Cancelación:**
    *   Si a 6 horas del inicio la partida no está llena (menos de 4 jugadores), se cancela automáticamente y se notifica a todos los apuntados.
    *   Si a menos de 6 horas del inicio un jugador abandona, la partida NO se cancela. Su plaza se libera y el slot vuelve a aparecer como "amarillo" para que un sustituto pueda unirse.
*   **FR-OPEN-04: Cambio de Organizador:** Si el creador original abandona la partida, el segundo usuario que se unió se convierte en el nuevo organizador.

### 3.4 Módulo de Lista de Espera

*   **FR-WAIT-01: Inscripción:** Si un slot está ocupado por una reserva privada, los usuarios pueden hacer clic y unirse a una lista de espera para ese slot.
*   **FR-WAIT-02: Proceso de Notificación:** Si la reserva se cancela, el sistema notifica automáticamente y solo al primer usuario de la lista de espera.
*   **FR-WAIT-03: Confirmación por Turno:** El usuario notificado tiene un tiempo limitado (configurable, 30 o 60 minutos) para confirmar la reserva a través de un enlace en el correo.
*   **FR-WAIT-04: Gestión de Turnos Expirados (Cron Job):** Un proceso automático en segundo plano (cron job) debe verificar los turnos expirados. Si un usuario no confirma a tiempo, su turno se marca como expirado y se notifica a la siguiente persona en la lista.

### 3.5 Módulo de Panel de Administración

*   Debe existir una interfaz web (admin.html) que permita al administrador realizar todas sus funciones de forma visual, incluyendo la gestión de usuarios, pistas, direcciones, bloqueos y ajustes generales.

## 4.0 Requisitos No Funcionales

*   **NF-01: Arquitectura:** La aplicación es una PWA con frontend en Vanilla JS, CSS y HTML, y un backend en Node.js/Express con una base de datos PostgreSQL.
*   **NF-02: Despliegue:** El sistema debe estar preparado para ser empaquetado y desplegado como contenedores Docker individuales y autocontenidos para cada comunidad residencial.
*   **NF-03: PWA:** La aplicación debe ser instalable en dispositivos móviles y de escritorio y ofrecer un funcionamiento básico offline (cacheo de la interfaz).

## 5.0 Nota sobre la Implementación con IA

Este documento representa un plano detallado y completo de la aplicación "Padel@Home". Es la "fuente de la verdad" que describe qué debe hacer el sistema y por qué.

Si bien herramientas como Google AI Studio son extremadamente potentes para generar ideas, texto, e incluso fragmentos de código, actualmente no tienen la capacidad de tomar un documento de especificaciones como este y generar automáticamente una aplicación full-stack completa, con su base de datos, backend, frontend, y la compleja interacción entre ellos.

Este documento sirve como la guía perfecta para que un desarrollador o un equipo de desarrollo (¡como hemos sido tú y yo!) construya la aplicación paso a paso, o para alimentar en el futuro a herramientas de IA de generación de código mucho más avanzadas.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Inciartej86)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Inciartej86)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
