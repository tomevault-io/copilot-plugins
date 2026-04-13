## gmb-playlist

> This is a simple Christmas-themed web application built with Python and Flask. It displays the lyrics of Christmas songs. The application reads song data from text files, making it easy to add new songs.

# Project Overview

This is a simple Christmas-themed web application built with Python and Flask. It displays the lyrics of Christmas songs. The application reads song data from text files, making it easy to add new songs.

## Key Technologies

*   **Backend:** Python, Flask
*   **Frontend:** HTML, CSS (with Jinja templating)

## Project Structure

*   `app.py`: The main Flask application file. It handles routing and loading song data.
*   `main.py`: A simple script, not directly used by the web application.
*   `pyproject.toml`: Defines project dependencies and metadata.
*   `songs/`: A directory containing the song lyrics in `.txt` files. Each file represents a song.
*   `templates/`: Contains the HTML templates for the web pages.
    *   `base.html`: The base template that other templates extend.
    *   `index.html`: The home page, displaying a list of available songs.
    *   `song.html`: The page for displaying the lyrics of a single song.
*   `static/`: Contains static assets.
    *   `css/style.css`: The stylesheet for the application.
    *   `logo.jpg`: A logo for the application.

## Building and Running

To run this application, you need to have Python and Flask installed.

1.  **Install Dependencies:**

    The project uses Flask. You can install it using pip:
    ```bash
    pip install Flask
    ```

2.  **Run the Application:**

    You can run the application in one of two ways:

    Using the `flask` command:
    ```bash
    export FLASK_APP=app.py
    flask run
    ```

    Or by directly running the Python script:
    ```bash
    python app.py
    ```

    The application will be available at `http://127.0.0.1:5000`.

## Development Conventions

*   **Adding a new song:** To add a new song, simply add a new `.txt` file to the `songs` directory. The name of the file (without the extension) will be used as the song title.
*   **Styling:** The application's styles can be modified in the `static/css/style.css` file.
*   **Templates:** The HTML structure of the pages can be modified in the `templates` directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/heikoliedtke)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/heikoliedtke)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
