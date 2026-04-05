# Fitbit Skill

This extension equips your AI assistant with a skill to access your Fitbit health data.

## Requirements

- A Fitbit Developer account with an Application registered with Fitbit (see [Registering an application](https://dev.fitbit.com/build/reference/web-api/developer-guide/getting-started/#Registering-an-Application))
- Allow your AI assistant to execute `curl` shell commands

## Installation

To install the extension, run the command:

Gemini
```
gemini extensions install https://github.com/sajalverma17/fitbit-skill
```

You will be prompted to configure your Fitbit-registered Application's details during installation. Find them in your [Fitbit Developer account](https://dev.fitbit.com/apps). These details are saved in an `.env` file within the extension's directory for future use.
 - `FITBIT_CLIENT_ID`
 - `FITBIT_CLIENT_SECRET`
 - `FITBIT_REDIRECT_URI`
 - `FITBIT_ACCESS_TOKEN`: Optional and can be a dummy value or left blank. The extension will set this after you allow it to access your Fitbit data.
 - `FITBIT_REFRESH_TOKEN`: Optional and can be a dummy value or left blank. The extension will set this after you allow it to access your Fitbit data.

If you update extension configuration later, you must restart your session.

## Usage

Ask your AI assistant for your Fitbit health data to activate this skill.

- "Show my steps for today."
- "How did I sleep last night?"
- "Get my average resting heart rate for this week."

On first use, you will be asked to open an authorization URL in your browser and grant AI assistant access to your Fitbit data. After that, the extension will update `FITBIT_ACCESS_TOKEN` and `FITBIT_REFRESH_TOKEN` in extension's `.env` file and re-use it across sessions.

## References
[Fitbit API Reference](https://dev.fitbit.com/build/reference/web-api/)

Note: Fitbit API will be deprecated and will migrate to Google Health API in September 2026.
