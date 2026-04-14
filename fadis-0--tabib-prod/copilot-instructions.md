## tabib-prod

> This file provides a comprehensive overview of the Tabibi project, designed to be understood and utilized by AI assistants and human developers alike.

# GEMINI.md - AI-Powered Project Documentation

This file provides a comprehensive overview of the Tabibi project, designed to be understood and utilized by AI assistants and human developers alike.

## Project Overview

Tabibi is a web application designed to help users find and book appointments with doctors. The application is built using a modern frontend stack, including:

*   **Framework:** React
*   **Build Tool:** Vite
*   **Language:** TypeScript
*   **Styling:** Tailwind CSS
*   **Routing:** React Router
*   **Backend:** Supabase (for database and authentication)

The application features a clean, component-based architecture with a clear separation of concerns. Key features include user authentication, a doctor search page, and a doctor dashboard.

## Building and Running

To get the project up and running locally, follow these steps:

1.  **Install Dependencies:**
    ```bash
    npm install
    ```

2.  **Set Environment Variables:**
    Create a `.env.local` file in the root of the project and add the following environment variable, as mentioned in the `README.md`:
    ```
    GEMINI_API_KEY=your_gemini_api_key
    ```

3.  **Run the Development Server:**
    ```bash
    npm run dev
    ```
    This will start the Vite development server, and you can view the application at `http://localhost:5173` (or another port if 5173 is in use).

4.  **Build for Production:**
    ```bash
    npm run build
    ```
    This command bundles the application for production into the `dist` directory.

5.  **Preview the Production Build:**
    ```bash
    npm run preview
    ```
    This command serves the production build locally for previewing.

## Development Conventions

*   **Component-Based Architecture:** The project is structured around reusable React components, located in the `src/components` directory.
*   **Functional Components and Hooks:** The codebase extensively uses functional components with React Hooks for state management and side effects.
*   **TypeScript:** The project is written in TypeScript, providing static typing for better code quality and maintainability.
*   **Lazy Loading:** Pages are lazy-loaded using `React.lazy` to improve initial load times.
*   **Styling:** Tailwind CSS is used for styling, with custom styles defined in `src/index.css`.
*   **Routing:** `react-router-dom` is used for client-side routing.
*   **State Management:** A combination of component state (`useState`, `useReducer`) and context (`AuthContext`, `LoaderProvider`) is used for state management.
*   **Backend Interaction:** The application interacts with a Supabase backend for data storage and user authentication, with the Supabase client initialized in `src/supabaseClient.ts`.

## Database Schema

```sql
-- Drop existing trigger if it exists
DROP TRIGGER IF EXISTS on_auth_user_created ON auth.users;

-- Drop existing function if it exists
DROP FUNCTION IF EXISTS public.handle_new_user();

-- Drop existing doctors table if it exists
DROP TABLE IF EXISTS doctors;
-- Drop existing specialties table if it exists
DROP TABLE IF EXISTS specialties;

-- Create a table for medical specialties
CREATE TABLE specialties (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL UNIQUE
);

-- Insert the specialties from your enum
INSERT INTO specialties (name) VALUES
('Cardiology'),
('Dermatology'),
('Neurology'),
('Orthopedics'),
('Pediatrics'),
('General Practice');


-- Create a table for doctors' public profiles
CREATE TABLE doctors (
   id uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
   name TEXT,
   specialty_id uuid REFERENCES specialties(id),
   rating NUMERIC(2, 1) CHECK (rating >= 0 AND rating <= 5),
   reviews INT,
   gender TEXT CHECK (gender IN ('Male', 'Female')),
   availability TEXT[],
   insurances TEXT[],
   address TEXT,
   lat NUMERIC(9, 6),
   lng NUMERIC(9, 6),
   bio TEXT,
   image_url TEXT,
   wilaya TEXT
);

-- Enable Row Level Security for the doctors table
ALTER TABLE doctors ENABLE ROW LEVEL SECURITY;


-- Policy: Public can read all doctor profiles
CREATE POLICY "Public can read all doctor profiles"
ON doctors
FOR SELECT
USING (true);

-- Policy: Doctors can insert their own profile
CREATE POLICY "Doctors can insert their own profile"
ON doctors
FOR INSERT
WITH CHECK (auth.uid() = id);

-- Policy: Doctors can update their own profile
CREATE POLICY "Doctors can update their own profile"
ON doctors
FOR UPDATE
USING (auth.uid() = id)
WITH CHECK (auth.uid() = id);

-- Function to create a new doctor profile when a new user signs up
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
 INSERT INTO public.doctors (id, name, image_url)
 VALUES (new.id, new.raw_user_meta_data->>'name', new.raw_user_meta_data->>'image_url');
 RETURN new;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Trigger to call the function when a new user is created
CREATE TRIGGER on_auth_user_created
 AFTER INSERT ON auth.users
 FOR EACH ROW EXECUTE PROCEDURE public.handle_new_user();
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fadis-0)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Fadis-0)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
