# berlingreen-app-config

Public OTA app configuration, served via **GitHub Pages** at `https://config.berlingreen.com/`.

`app-config.json` is fetched by the BerlinGreen mobile app's **"Check for Firmware Update"** flow
(`src/ota/config.ts` → `fetchOtaConfig`/`findOffer`). It decides which firmware update (if any) a box
is **offered** — by `FWVer` byte, channel, and beta-enrollment. **The box never reads this file**
(the firmware fetches `firmware.json` + the binaries from the pinned OTA server instead).

## No secrets / no PII
- Beta enrollment is gated by a Firebase **custom claim** (`betaEnrolled`) in each user's token —
  **not** a list in this file.
- Firmware binaries and `firmware.json` live on the pinned OTA server, not here.

## Editing the rollout (no app release)
Edit `app-config.json` and push to `main`. Pages redeploys; the app picks it up on the next check.

- `offers[]` — per source `FWVer` byte: which channel/version to offer, and `enrolledOnly`.
  - `{ "fromByte": 3, … }` → 2.0.3 boxes (app routes to the WiFi flow).
  - `{ "fromByte": 4, … }` → 4.x boxes (app routes to the BLE U/C flow).
- `registry` / `channels` — informational today; reserved for server-driven decode/display later.

Served from this repo's root via Pages; `CNAME` pins the custom domain `config.berlingreen.com`.
