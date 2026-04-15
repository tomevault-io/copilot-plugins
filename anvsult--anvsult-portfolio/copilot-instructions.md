## anvsult-portfolio

> This is a personal portfolio website built with Next.js, a popular React framework. The project showcases the developer's skills, projects, and testimonials. It also includes a contact form for communication. The website is designed to be bilingual (English and French), with internationalization handled by `next-intl`.

# Project Overview

This is a personal portfolio website built with Next.js, a popular React framework. The project showcases the developer's skills, projects, and testimonials. It also includes a contact form for communication. The website is designed to be bilingual (English and French), with internationalization handled by `next-intl`.

## Key Technologies

*   **Framework:** Next.js (React)
*   **Styling:** Tailwind CSS with Radix UI and custom components.
*   **Database:** PostgreSQL managed with Prisma ORM.
*   **Authentication:** Supabase is used for user authentication.
*   **Internationalization:** `next-intl` is used for English and French language support.

## Project Structure

The project follows a standard Next.js `app` directory structure.

*   `src/app/[locale]/`: Contains the main pages of the application, with a dynamic `[locale]` segment for internationalization.
*   `src/components/`: Contains reusable React components, organized into `portfolio/` and `ui/` subdirectories.
*   `src/lib/`: Contains utility functions, database client (`db.ts`), and Supabase client setup.
*   `prisma/`: Contains the Prisma schema (`schema.prisma`) which defines the database models.
*   `messages/`: Contains the translation files (`en.json`, `fr.json`) for `next-intl`.
*   `public/`: Contains static assets like images and icons.

## Building and Running

### Prerequisites

*   Node.js and npm (or yarn/pnpm)
*   A PostgreSQL database

### Setup

1.  **Install dependencies:**
    ```bash
    npm install
    ```
2.  **Set up environment variables:**
    Create a `.env` file in the root of the project and add the following variables:
    ```
    DIRECT_URL="your-postgresql-database-url"
    NEXT_PUBLIC_SUPABASE_URL="your-supabase-url"
    NEXT_PUBLIC_SUPABASE_ANON_KEY="your-supabase-anon-key"
    ```
3.  **Run database migrations:**
    ```bash
    npx prisma db push
    ```

### Development

To run the development server:
```bash
npm run dev
```
The application will be available at `http://localhost:3000`.

### Building for Production

To create a production build:
```bash
npm run build
```

To start the production server:
```bash
npm run start
```

## Development Conventions

*   **Styling:** The project uses Tailwind CSS for styling. Utility classes are preferred over custom CSS.
*   **Components:** Reusable components are created in the `src/components` directory.
*   **Database:** The database schema is managed with Prisma. Any changes to the schema should be made in `prisma/schema.prisma` and then migrated to the database.
*   **Internationalization:** All user-facing text should be added to the `messages/en.json` and `messages/fr.json` files and accessed using the `useTranslations` hook from `next-intl`.
*   **Linting:** The project uses ESLint for code linting. Run `npm run lint` to check for linting errors.

## Supabase Storage for Resume

To serve the resume PDF efficiently, we use Supabase Storage. Here's how to configure it:

### 1. Create a Public Bucket

1.  Go to your Supabase project dashboard.
2.  Navigate to the **Storage** section from the sidebar.
3.  Click on **Create a new bucket**.
4.  Enter a name for your bucket (e.g., `resumes`).
5.  Toggle the **Public bucket** option to **On**. This allows direct public access to the files.
6.  Click **Create bucket**.

### 2. Upload Your Resume

1.  Once the bucket is created, click on it to open it.
2.  You can drag and drop your resume PDF file into the bucket or use the **Upload file** button.

### 3. Get the Public URL

1.  After the upload is complete, click on the uploaded file.
2.  In the details panel that appears, you will find a **Get URL** button. Click it to copy the public URL of your file. It will look something like this: `https://<your-project-ref>.supabase.co/storage/v1/object/public/resumes/your-resume.pdf`.

### 4. Add the URL to Your Database

The application fetches the resume URL from the `Resume` table in the database. You need to add an entry for each locale.

1.  **Open Prisma Studio:**
    ```bash
    npx prisma studio
    ```
2.  In Prisma Studio, navigate to the `Resume` model.
3.  Click **Add new record**.
4.  Fill in the fields:
    *   `fileUrl`: Paste the public URL you copied from Supabase.
    *   `fileName`: The name you want the file to have when downloaded (e.g., `Anvar_Sultanov_Resume.pdf`).
    *   `locale`: The language of the resume (`en` or `fr`).
    *   `version`: A version number for the resume (e.g., `1.0`).
5.  Click **Save 1 change**.
6.  Repeat for other locales if you have different resume files for each language.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anvsult)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/anvsult)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
