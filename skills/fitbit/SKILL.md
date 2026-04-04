---
name: fitbit
description: Use when user asks to access their Fitbit activity, sleep, heart rate, or other health metrics. Authorizes with their Fitbit account and gets data from Fitbit API.
---
# Fitbit API
Use `curl` to fetch data from the Fitbit REST API.

## API Usage
Read FITBIT_ACCESS_TOKEN from `fitbit.json` and use it to make authorized requests.
If unavailable or a request returns `401`, see Authorization section.

Examples:

Get activity data for a specific date:
```bash
curl "https://api.fitbit.com/1/user/-/activities/date/today.json" -H "Authorization: Bearer FITBIT_ACCESS_TOKEN"
```

Get sleep data for a specific date:
```bash
curl "https://api.fitbit.com/1/user/-/sleep/date/2024-01-01.json" -H "Authorization: Bearer FITBIT_ACCESS_TOKEN"
```

- Dates must be formatted as `YYYY-MM-DD` or use `today` for the current date.
- Response units depend on the `Accept-Language` header: `en_US` for imperial, `en_GB` for UK, anything else defaults to metric.
- Rate limit is 150 requests per hour per user. On `429`, read `Fitbit-Rate-Limit-Reset` response header for seconds to wait before retrying.

Refer to API reference for endpoints, data and unit types: https://dev.fitbit.com/build/reference/web-api.
Refer to Developer Guide for best practices and rate limits: https://dev.fitbit.com/build/reference/web-api/developer-guide/application-design

## Authorization
Use OAuth 2.0 Authorization Code Grant Flow.

Read authorization credentials from `fitbit.json` and keep in memory for the session:
- FITBIT_CLIENT_ID
- FITBIT_CLIENT_SECRET
- FITBIT_REDIRECT_URI

Scopes available:
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
1. Select scopes based on user request and construct Authorization URL using this template: https://www.fitbit.com/oauth2/authorize?response_type=code&client_id=FITBIT_CLIENT_ID&redirect_uri=FITBIT_REDIRECT_URI&scope=SELECTED_SCOPES. Scopes must be space-separated and URL-encoded (use `%20`).
2. Send Authorization URL to the user, ask them to open it in a browser to authorize the app, and then paste back `code` parameter value from browser's address bar.
3. Exchange the code for tokens with POST request to `https://api.fitbit.com/oauth2/token` with:
   - `Authorization` header containing base64-encoded FITBIT_CLIENT_ID and FITBIT_CLIENT_SECRET
   - `Content-Type` header set to `application/x-www-form-urlencoded`
   - Request body containing `grant_type=authorization_code`, `code=USER_PROVIDED_CODE` and `redirect_uri=FITBIT_REDIRECT_URI`
4. Save access_token and refresh_token as FITBIT_ACCESS_TOKEN and FITBIT_REFRESH_TOKEN in `fitbit.json` and update in memory for the session.

### Token Refresh Workflow
If any API call returns 401 or access token is known to be expired, run this workflow silently without prompting the user.

1. Read FITBIT_REFRESH_TOKEN from `fitbit.json`.
2. Obtain fresh pair of tokens with POST request to `https://api.fitbit.com/oauth2/token` with:
   - `Authorization` header containing base64-encoded FITBIT_CLIENT_ID and FITBIT_CLIENT_SECRET
   - `Content-Type` header set to `application/x-www-form-urlencoded`
   - Request body containing `grant_type=refresh_token` and `refresh_token=FITBIT_REFRESH_TOKEN`
3. Save access_token and refresh_token as FITBIT_ACCESS_TOKEN and FITBIT_REFRESH_TOKEN in `fitbit.json` and update in memory for the session. Refresh tokens are single-use.

If this fails with `invalid_grant`, run Initial Authorization Workflow.


