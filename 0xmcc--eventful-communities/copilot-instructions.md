## eventful-communities

> *   **Project Name:** Vently

# Cursor Rules for Vently Project

## Project Overview

*   **Project Name:** Vently

*   **Description:** Vently is a dynamic, mobile-friendly platform designed to centralize event promotion and engagement within communities. It features a user-friendly interface that allows users to discover events through a map, engage with communities, purchase tickets, and participate in discussions.

*   **Tech Stack:**

    *   Frontend: ReactJS
    *   Styling: Tailwind CSS
    *   Backend & Database: Supabase
    *   Payment Integration: Stripe
    *   Location Services: Google Maps API

*   **Key Features:**

    *   Interactive event map with clickable pins for event discovery.
    *   Event and community management capabilities.
    *   Secure ticket purchasing with payment gateway integration.
    *   User roles for Regular Users, Event Organizers, and Community Admins.

## Project Structure

*   **Root Directory:** Contains the main configuration files and documentation.

*   `/frontend/`: All frontend-related code.

    *   `/components/`:

        *   `MapView`: Displays events on a map.
        *   `EventCard`: Presents event details.
        *   `CommunityList`: Displays a list of communities.
        *   Other UI components as necessary.

    *   `/assets/`:

        *   Event images and icons.
        *   Community profile pictures.

    *   `/styles/`:

        *   Global styles utilizing Tailwind CSS.

*   `/backend/`: All backend-related code.

    *   `/controllers/`: Handles CRUD operations for events and community entities.
    *   `/models/`: Schema definitions for Events, Tickets, Users, and Communities.
    *   `/routes/`: API endpoints for frontend-backend communication.

*   `/config/`: Configuration files for environment variables and application settings.

*   `/tests/`: Unit and integration tests for frontend and backend modules.

## Development Guidelines

*   **Coding Standards:**

    *   Follow the Airbnb React/JSX Style Guide for JavaScript and React components.
    *   Consistent use of `eslint` for styling and error prevention.
    *   Component files should be organized by feature or route for clarity.

*   **Component Organization:**

    *   Each component should be modular, maintaining a single responsibility principle.
    *   Utilize context API or third-party state management tools where appropriate.

## Cursor IDE Integration

*   **Setup Instructions:**

    *   Clone the repository locally.
    *   Run `npm install` to set up dependencies.
    *   Start the development environment with `npm start` for frontend and backend servers.

*   **Key Commands:**

    *   `npm run test` to run tests.
    *   Use the built-in tools of Cursor AI for real-time code suggestions and modifications.

## Additional Context

*   **User Roles:**

    *   Regular Users: Can browse events and communities, RSVP, and purchase tickets.
    *   Event Organizers: Can create and manage events, access to event-related analytics (future feature).
    *   Community Admins: Manage community settings and moderate content.

*   **Accessibility Considerations:**

    *   Use ARIA roles to enhance component accessibility.
    *   Ensure color contrast meets WCAG standards for the synthe purple theme.

---
> Source: [0xmcc/eventful-communities](https://github.com/0xmcc/eventful-communities) — distributed by [TomeVault](https://tomevault.io/claim/0xmcc).
<!-- tomevault:4.0:copilot_instructions:2026-04-18 -->
