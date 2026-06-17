## tweetclaw

> Post tweets and manage Twitter/X account via API v2. Use when the user asks to tweet, post updates, or interact with Twitter.


# TweetClaw Skill

This skill allows OpenClaw to post tweets using the Twitter API v2 via a Node.js script.

## Setup

1.  **Create a Twitter Developer Account**: Go to [developer.twitter.com](https://developer.twitter.com) and create a project/app.
2.  **Get Credentials**: Obtain the following keys and tokens from the Developer Portal (ensure "Read and Write" permissions are enabled for the Access Token):
    *   API Key
    *   API Key Secret
    *   Access Token
    *   Access Token Secret
3.  **Configure Environment**:
    Create a `.env` file in `~/Agents/tweetclaw/.env` with the following content:

    ```bash
    TWITTER_API_KEY=your_api_key
    TWITTER_API_SECRET=your_api_key_secret
    TWITTER_ACCESS_TOKEN=your_access_token
    TWITTER_ACCESS_SECRET=your_access_token_secret
    ```

## Usage

### Posting a Tweet

To post a tweet, use the provided script:

```bash
cd ~/Agents/tweetclaw
node scripts/post_tweet.js "Your tweet text here"
```

### Troubleshooting

-   **Error 403**: Ensure your app has "Read and Write" permissions in the Twitter Developer Portal. If you changed permissions, regenerate your Access Tokens.
-   **Error 401**: Check that your API Key/Secret and Access Token/Secret are correct in `.env`.

## Future Expansion

-   Add scripts to read timeline (GET /2/tweets/search/recent).
-   Add scripts to reply or like tweets.

---
> Source: [saprative/tweetclaw](https://github.com/saprative/tweetclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
