## fit-ai

> This project, "FitAI Student Planner," is a personalized fitness and diet assistant built with Python and Streamlit. It's designed specifically for Indian students, taking into account their unique dietary habits, budget constraints, and lifestyles. The application leverages Google's Gemini AI to provide personalized meal plans, workout routines, and calorie tracking from food images.

# GEMINI.md

## Project Overview

This project, "FitAI Student Planner," is a personalized fitness and diet assistant built with Python and Streamlit. It's designed specifically for Indian students, taking into account their unique dietary habits, budget constraints, and lifestyles. The application leverages Google's Gemini AI to provide personalized meal plans, workout routines, and calorie tracking from food images.

**Main Technologies:**

*   **Framework:** Streamlit
*   **AI:** Google Gemini (via `google-generativeai`)
*   **Data Handling:** JSON for user profiles and logs.
*   **Data Visualization:** Plotly and Pandas
*   **Environment Management:** `python-dotenv`

**Architecture:**

*   `main.py`: The main application entry point and dashboard.
*   `pages/`: A directory containing the different pages of the Streamlit application, such as user profile, meal plan, workout plan, etc.
*   `ai_engine.py`: A dedicated module for all interactions with the Google Gemini API. It contains functions for generating meal plans, workout routines, analyzing food images, and a general AI chat.
*   `utils.py`: A set of utility functions for handling data, such as loading and saving user profiles and logs, and creating data visualizations.
*   `data/`: A directory that stores user data, including `user_profile.json` and `logs.json`.
*   `.env`: A file to store the `GEMINI_API_KEY`.

## Building and Running

1.  **Install Dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

2.  **Set up Environment Variables:**
    Create a `.env` file in the root directory and add your Gemini API key:
    ```
    GEMINI_API_KEY="YOUR_API_KEY"
    ```

3.  **Run the Application:**
    ```bash
    streamlit run main.py
    ```
    The application will be available at `http://localhost:8501`.

## Development Conventions

*   **Styling:** The application uses a custom dark theme with CSS injected via `st.markdown`.
*   **Data Storage:** User data is stored in simple JSON files in the `data/` directory.
*   **AI Integration:** All AI-related functionalities are centralized in the `ai_engine.py` module.
*   **Modularity:** The application is divided into multiple pages using Streamlit's multi-page feature, with each page having its own Python file in the `pages/` directory.
*   **Utility Functions:** Common functions are kept in `utils.py` to be reused across the application.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ssmadhavan006)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ssmadhavan006)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
