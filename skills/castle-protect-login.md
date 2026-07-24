---
name: Protect a login with Castle Risk API
description: Score an authenticated login event in real time and act on the risk verdict (allow / challenge / deny).
api: https://api.castle.io
operations:
  - "POST /v1/risk"
---

# Protect a login with Castle

Use Castle's **Risk API** to score a login **after** the user's credentials
verify, and let the returned policy action decide whether to allow, challenge,
or deny.

## Prerequisites
- Backend authenticates to `https://api.castle.io` with HTTP Basic auth: the
  **API Secret** as the password, empty username (`authentication/castle-authentication.yml`).
- The client page/app runs the browser or mobile SDK and produces a
  `request_token` via `castle.createRequestToken()`. The token expires after
  **120 seconds** and is single-use — fetch a fresh one on every backend call
  (`conventions/castle-conventions.yml`).

## Steps
1. On the client, generate a fresh `request_token` immediately before the login
   request and send it to your backend.
2. After the password check succeeds, call `POST /v1/risk` with:
   - `type: "$login"`, `status: "$succeeded"`
   - `request_token`
   - `user` (id, email, traits)
   - `context` (ip, headers)
3. Read the response `policy.action`:
   - `allow` — complete the login.
   - `challenge` — trigger step-up (2FA / verification), then send a
     `$challenge` event.
   - `deny` — block and log out.
4. Handle errors per `errors/castle-problem-types.yml` (401 bad secret, 422
   expired request token, 429 rate limit).

## Testing
In a Sandbox environment use a Mock Request Token to force outcomes, e.g.
`test|device:bot_on_linux|ip:jp` (`sandbox/castle-sandbox.yml`).
