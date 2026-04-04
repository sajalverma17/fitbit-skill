# Fitbit Skill

Equips your AI assistant to work with your Fitbit health data.

## Requirements

- An Application registered with Fitbit (see [Registering an application](https://dev.fitbit.com/build/reference/web-api/developer-guide/getting-started/#Registering-an-Application))
- `fitbit.json` file created in your working directory with your Application's Client ID, Client Secret and Redirect URI:
    ```json
    {
        "FITBIT_CLIENT_ID": "your-client-id",
        "FITBIT_CLIENT_SECRET": "your-client-secret",
        "FITBIT_REDIRECT_URI": "your-redirect-uri",
        "FITBIT_ACCESS_TOKEN": null,
        "FITBIT_REFRESH_TOKEN": null
    }
    ```
- AI assistant permission to execute `curl` shell command and read/write access to `fitbit.json`

## Usage

Ask your AI assistant for your Fitbit health data to activate this skill.

- "Show my steps for today."
- "How did I sleep last night?"
- "Get my average resting heart rate for this week."

On first use, you will be asked to open an authorization URL in your browser to grant your AI assistant access to your Fitbit data. After that, `FITBIT_ACCESS_TOKEN` and `FITBIT_REFRESH_TOKEN` are saved to `fitbit.json` automatically and reused for future chats.

## References
[Fitbit API Reference](https://dev.fitbit.com/build/reference/web-api/)

Note: Fitbit API will be deprecated and will migrate to Google Health API in September 2026.
