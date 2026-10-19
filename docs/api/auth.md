# Sessions

Every route except the three listed under "public" needs a session. A session is a token from `/api/auth/login`, sent as the `X-Auth-Token` header or the `ie_tok` cookie. The WebSocket takes it as the `tok` query parameter.

| Item | Value |
|---|---|
| First start | No password exists. `/api/auth/state` reports `provisioned: false` and the board serves the setup page until `/api/auth/password` sets one. There is no default password. |
| Public routes | `GET /api/auth/state`, `POST /api/auth/login`, `POST /api/auth/password` while unprovisioned |
| Password storage | Salted PBKDF2-SHA256, 10,000 iterations, in the board's settings flash. A login takes 3 to 5 s. |
| Token | 128-bit random, 8 concurrent sessions, 24 h sliding idle timeout, kept across reboots and updates |
| Lockout | 5 free failed attempts, then a wait of 30 s that doubles per failure up to 15 min. `/api/auth/state` and the login answer carry `lock` in seconds. |
| Change | `POST /api/auth/password {"new","old"}` ends every session |
| Logout | `POST /api/auth/logout` ends the calling session |
| Log | `GET /api/auth/log`: the last sign-in attempts with the uptime at which they happened |
| Recovery | Factory reset by holding the user button, or `POST /api/factory-reset {"confirm":true}` from a session. Both clear the password, the stores and the saved network. |

| Route | Method | Body | Answer |
|---|---|---|---|
| /api/auth/state | GET | none | `{provisioned, authed, lock}` |
| /api/auth/login | POST | `{"pass"}` | `{ok, token}` and the `ie_tok` cookie, or `{ok:false, lock}` |
| /api/auth/password | POST | `{"new"}` while unprovisioned, `{"new","old"}` afterwards | `{ok}` |
| /api/auth/logout | POST | none | `{ok}` |
| /api/auth/log | GET | none | `[{up, msg}]` |
