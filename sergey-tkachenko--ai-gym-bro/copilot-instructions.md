## ai-gym-bro

> This rule outlines how the AI Gym Bro bot handles its configuration, data persistence, and logging.

# Configuration, Persistence, and Logging

This rule outlines how the AI Gym Bro bot handles its configuration, data persistence, and logging.

## Configuration

*   The bot primarily uses environment variables for configuration. These are loaded using `dotenv` from a `.env` file in the project root during startup (as seen in [ai_gym_bro/main.py](mdc:ai_gym_bro/main.py)).
*   Key environment variables include:
    *   `TELEGRAM_BOT_TOKEN`: Essential for connecting to the Telegram Bot API.
    *   `OPENAI_API_KEY`: Required for interacting with the OpenAI API via the [ai_gym_bro/services/openai_service.py](mdc:ai_gym_bro/services/openai_service.py).
*   The absence of these critical variables is logged as an error.

## Persistence

*   To maintain conversation state across bot restarts (e.g., during deployments or unexpected shutdowns), the bot uses `PicklePersistence` from the `python-telegram-bot` library.
*   The persistence file is located at [ai_gym_bro/persistence/bot_persistence.pkl](mdc:ai_gym_bro/persistence/bot_persistence.pkl).
*   The directory for this file ([ai_gym_bro/persistence/](mdc:ai_gym_bro/persistence)) is created automatically if it doesn't exist.
*   `context.user_data` is the primary dictionary persisted, containing all collected user information, the generated plan, and conversation history for refinement.

## Logging

*   The bot utilizes the **Loguru** library for flexible and powerful logging.
*   Configuration is done in the `if __name__ == "__main__":` block of [ai_gym_bro/main.py](mdc:ai_gym_bro/main.py).
*   **File Logging**: Debug level messages and above are logged to [ai_gym_bro.log](mdc:ai_gym_bro.log) with rotation (10 MB file size).
The format includes timestamp, level, module, function, line number, and message.
*   **Console Logging**: Info level messages and above are printed to the console with a colorized, more readable format.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sergey-Tkachenko)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sergey-Tkachenko)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
