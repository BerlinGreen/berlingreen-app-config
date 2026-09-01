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
