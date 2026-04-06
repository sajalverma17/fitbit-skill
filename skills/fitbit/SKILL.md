---
name: fitbit
description: Use when the user asks to access their Fitbit activity, sleep, heart rate, or other health metrics. Authenticates with the Fitbit Web API and retrieves data using credentials provided during installation.
---
# Fitbit API
Use `curl` to retrieve data from the Fitbit Web API.

**IMPORTANT**: All environment variables referenced in the skill must be read from, and updated in the .env file present at `~/.gemini/extensions/fitbit-skill/.env` for the skill to work across sessions.

The .env file is automatically created with three required authentication credentials: FITBIT_CLIENT_ID, FITBIT_CLIENT_SECRET, FITBIT_REDIRECT_URI provided by the user when installing the skill. You are expected to automatically add or update FITBIT_ACCESS_TOKEN and FITBIT_REFRESH_TOKEN in the .env file after successful authorization by the user or token refresh. If the file is missing or does not contain the three required authentication credentials, prompt the user to provide them and create the file at the specified location before proceeding with this skill.

## API Usage
Read FITBIT_ACCESS_TOKEN environment variable to make authorized requests.
If FITBIT_ACCESS_TOKEN is not found or a request returns `401`, you must help the user authenticate or refresh the tokens as described in the Authorization section.

Example requests:

Get activity data for a specific date:
```bash
curl "https://api.fitbit.com/1/user/-/activities/date/today.json" -H "Authorization: Bearer <FITBIT_ACCESS_TOKEN>"
```

Get sleep data for a specific date:
```bash
curl "https://api.fitbit.com/1/user/-/sleep/date/2024-01-01.json" -H "Authorization: Bearer <FITBIT_ACCESS_TOKEN>"
```

- Dates must be formatted as `YYYY-MM-DD` or use `today` for the current date.
- Response units depend on the `Accept-Language` header: `en_US` for imperial, `en_GB` for UK, anything else defaults to metric.
- Rate limit is 150 requests per hour per user. On `429`, read `Fitbit-Rate-Limit-Reset` response header for seconds to wait before retrying.

Refer to API reference for endpoints, data and unit types: https://dev.fitbit.com/build/reference/web-api.
Refer to Developer Guide for best practices and rate limits: https://dev.fitbit.com/build/reference/web-api/developer-guide/application-design

## Authorization
Use OAuth 2.0 Authorization Code Grant Flow to authorize with Fitbit API with below available scopes:

- `activity`: Steps, distance, calories, etc.
- `heartrate`: Heart rate data.
- `location`: GPS and location data.
- `nutrition`: Food and water logs.
- `oxygen_saturation`: SpO2 data.
- `profile`: User profile info.
- `respiratory_rate`: Breathing rate data.
- `settings`: User device settings.
- `sleep`: Sleep logs and stages.
- `social`: Friends and leaderboard.
- `temperature`: Core and skin temperature.
- `weight`: Weight and body fat logs.

Reference: https://dev.fitbit.com/build/reference/web-api/developer-guide/authorization/

### Initial Authorization Workflow
1. Select scopes based on user request, read FITBIT_CLIENT_ID, FITBIT_CLIENT_SECRET, FITBIT_REDIRECT_URI environment variables, and construct Authorization URL using this template: https://www.fitbit.com/oauth2/authorize?response_type=code&client_id=<FITBIT_CLIENT_ID>&redirect_uri=<FITBIT_REDIRECT_URI>&scope=<SELECTED_SCOPES>. Scopes must be space-separated and URL-encoded (use `%20`).
2. Send Authorization URL to the user, ask them to open it in a browser to authorize the app, and then paste back `code` parameter value from browser's address bar.
3. Exchange the code for tokens with POST request to `https://api.fitbit.com/oauth2/token` with:
   - `Authorization` header containing the base64-encoded string of FITBIT_CLIENT_ID:FITBIT_CLIENT_SECRET
   - `Content-Type` header set to `application/x-www-form-urlencoded`
   - Request body containing `grant_type=authorization_code`, `code=<user-provided code>` and `redirect_uri=<FITBIT_REDIRECT_URI>`
4. Update the FITBIT_ACCESS_TOKEN and FITBIT_REFRESH_TOKEN environment variables in .env file with values obtained from response for usage across sessions.

### Token Refresh Workflow
If any API call returns 401 or access token is known to be expired, run this workflow silently without prompting the user.

1. Read FITBIT_REFRESH_TOKEN environment variable. If it is unavailable, run the Initial Authorization Workflow.
2. Obtain a fresh pair of tokens with POST request to `https://api.fitbit.com/oauth2/token` with:
   - `Authorization` header containing the base64-encoded string of FITBIT_CLIENT_ID:FITBIT_CLIENT_SECRET
   - `Content-Type` header set to `application/x-www-form-urlencoded`
   - Request body containing `grant_type=refresh_token` and `refresh_token=<FITBIT_REFRESH_TOKEN>`
3. Update the FITBIT_ACCESS_TOKEN and FITBIT_REFRESH_TOKEN environment variables in .env file with values obtained from response for usage across sessions.

If this fails with `invalid_grant`, run the Initial Authorization Workflow.
