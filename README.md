# berlingreen-app-config

Static host for the BerlinGreen mobile app, served by GitHub Pages at
**https://app.berlingreen.com**.

The app fetches `app-config.json` from here to decide which firmware update, if any, to offer a
box. To change what is offered, edit `app-config.json` and push to `main`.

## Why its own subdomain

berlingreen.com is a Shopify store. Shopify owns `/.well-known/` on that domain and serves stubs
there that merchants cannot change, so the app can never be associated with the apex. This host is
ours, and because nothing else lives on it the app can claim the whole host without ever capturing
a shop URL.

Previously served at config.berlingreen.com; that name still points at GitHub Pages, which
redirects it here.

## What it serves

| Path | Used by |
|---|---|
| `/app-config.json` | The app's OTA offer manifest (`src/ota/config.ts` → `OTA_CONFIG_URL`) |
| `/.well-known/assetlinks.json` | Android App Links (`delegate_permission/common.handle_all_urls`) |
| `/.well-known/apple-app-site-association` | iOS Universal Links |

The app claims this host in `AndroidManifest.xml`, so once Android has verified the association,
**every** `https://app.berlingreen.com/...` URL tapped on a phone with the app installed opens the
app rather than a browser. Any path published here therefore needs a matching route in the app's
`linking` config (`src/App.js`) and, for people without the app, a page that a browser can render.

## `.nojekyll` is load-bearing

GitHub Pages runs Jekyll, which skips files and directories whose names begin with a dot —
`.well-known/` would silently never publish. The empty `.nojekyll` file at the repo root turns
Jekyll off. Do not delete it.

## Fingerprints in `assetlinks.json`

- `D8:32:0C:…` — the Play **app signing** key (Play Console → Protected with Play → App signing).
  Play re-signs every upload with it, so this is the certificate actually on users' devices and the
  one that matters in production.
- `BC:49:C0:…` — the current **upload** certificate, so locally assembled release builds and
  internal app sharing match too. Re-derive it after any upload-key reset:

  ```sh
  keytool -export -rfc -keystore android/app/berlin-green-key.keystore \
    -alias berlin-green-key-alias -file /tmp/upload_certificate.pem
  openssl x509 -in /tmp/upload_certificate.pem -noout -fingerprint -sha256
  ```

Never add the Android debug fingerprint: it is shared by every machine that has ever run
`assembleDebug`, so any app signed with it would satisfy the association.

## Verifying

```sh
curl -sI https://app.berlingreen.com/app-config.json        # 200, application/json
curl -s  https://app.berlingreen.com/.well-known/assetlinks.json | jq

# what Google's crawler sees (empty errorCode == working)
curl -s "https://digitalassetlinks.googleapis.com/v1/statements:list\
?source.web.site=https://app.berlingreen.com\
&relation=delegate_permission/common.handle_all_urls" | jq

# what Apple's CDN fetched and parsed — the only iOS check that counts, since GitHub Pages
# cannot be told to send application/json for the extensionless AASA file
curl -s "https://app-site-association.cdn-apple.com/a/v1/app.berlingreen.com" | jq
```

Both files must be reachable over HTTPS with no redirect; Google and Apple refuse to follow one.
