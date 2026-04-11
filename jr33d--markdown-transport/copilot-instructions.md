## markdown-transport

> This project is a monorepo containing the `markdown-transport` library and an example application that uses it.

# Project: Markdown Transport

## Project Overview

This project is a monorepo containing the `markdown-transport` library and an example application that uses it.

The `markdown-transport` library is a TypeScript-based utility for reading, parsing, and processing Markdown files. It is designed to be a reusable "transport layer" for content that can be integrated into various applications, such as static site generators (Next.js, Astro), blogs, or any project that relies on Markdown-based content.

The library handles:
- Reading Markdown files from the filesystem or a remote Git repository.
- Parsing YAML frontmatter (e.g., title, slug, publishDate, draft status, tags).
- Filtering posts based on draft status and publish date.
- Sorting posts by date.

The example application demonstrates how to use the `markdown-transport` library to load, filter, and display posts from a `content` directory or a remote Git repository.

## Building and Running

### Library (`markdown-transport`)

- **Build:** To compile the TypeScript library to JavaScript, run the following command from the `markdown-transport` directory:
  ```bash
  npm run build
  ```
  This will generate the output in the `markdown-transport/dist` directory.

### Example App (`example-app`)

- **Run:** To run the example application, which uses the `markdown-transport` library, execute the following command from the `example-app` directory:
  ```bash
  npm start
  ```
  This will run the `src/index.ts` script using `ts-node`, which loads posts from the `content` directory and prints them to the console.

### Loading Content from a Git Repository

The `example-app` can be configured to load content from a remote Git repository instead of the local `content` directory. This is useful for decoupling the content from the application code.

To use this feature:

1.  **Create a `.env` file:** In the `example-app` directory, create a copy of the `.env.example` file and name it `.env`.
2.  **Set the Git repository URL:** In the `.env` file, set the `CONTENT_GIT_REPO_URL` variable to the URL of the Git repository you want to clone.
    ```
    CONTENT_GIT_REPO_URL=https://github.com/user/repository.git
    ```
3.  **Configure Git Strategy (Optional):**
    *   `GIT_STRATEGY`: Controls how the Git repository is handled.
        *   `clone` (default): Always performs a fresh clone of the repository.
        *   `pull`: If the repository already exists locally, it will pull the latest changes. If not, it will clone it.
        *   `none`: Skips all Git operations. Content will be loaded from `GIT_LOCAL_PATH` if specified, or `contentDir`.
    ```
    GIT_STRATEGY=pull
    ```
4.  **Configure Local Git Path (Optional):**
    *   `GIT_LOCAL_PATH`: Specifies a persistent local directory to store the cloned Git repository. If not provided, a temporary directory will be used.
    ```
    GIT_LOCAL_PATH=./content-repo
    ```
5.  **Configure Git Author Information (Optional):**
    *   `GIT_AUTHOR_NAME`: The name to use for Git authoring (e.g., for merges during pull).
    *   `GIT_AUTHOR_EMAIL`: The email to use for Git authoring.
    ```
    GIT_AUTHOR_NAME="Your Name"
    GIT_AUTHOR_EMAIL="your.email@example.com"
    ```
6.  **Run the application:** Start the application as usual:
    ```bash
    npm start
    ```
    The application will now clone or pull the specified Git repository into the configured directory and load the content from there.

#### Syncing Content

The `MarkdownTransport` class now includes a `sync()` method that can be called explicitly to pull the latest changes from the remote Git repository. This is useful for updating content without restarting the application or performing a full re-clone.

Example usage in `example-app/src/index.ts`:

```typescript
// ...
if (gitRepoUrl && gitLocalPath) {
  console.log('Syncing Git repository...');
  await transport.sync();
  console.log('Git repository synced.\n');
}
// ...
```

#### Private Repositories

For private repositories, you'll need to provide authentication credentials. You can do this by setting the `GIT_USERNAME` and `GIT_PASSWORD` environment variables in your `.env` file.

-   `GIT_USERNAME`: Your Git username.
-   `GIT_PASSWORD`: Your Git password or a personal access token (PAT). Using a PAT is recommended for better security.

Example `.env` file for a private repository:

```
CONTENT_GIT_REPO_URL=https://github.com/user/private-repo.git
GIT_USERNAME=your-username
GIT_PASSWORD=your-password-or-pat
```

### Root

From the root of the project, you can run the following commands:

- **Build Library:**
  ```bash
  npm run build:lib
  ```
- **Build All:**
  ```bash
  npm run build
  ```

## Development Conventions

- **TypeScript:** The entire project is written in TypeScript, providing type safety and better developer experience.
- **Monorepo:** The project is structured as a monorepo, with the core library and the example application in separate packages.
- **Local Dependencies:** The example application uses a local `file:` dependency to link to the `markdown-transport` library, which is a common practice for monorepo development.
- **Environment Variables:** The `example-app` uses a `.env` file to manage environment variables. This allows for easy configuration of the content source (local or Git repository) and authentication credentials.
- **Frontmatter:** The library uses the `gray-matter` package to parse YAML frontmatter from Markdown files. The following fields are officially supported: `title`, `slug`, `summary`, `publishDate`, `tags`, and `draft`.
- **ES Modules:** Both the library and the example app use ES Modules (`import`/`export` syntax).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JR33D)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JR33D)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
