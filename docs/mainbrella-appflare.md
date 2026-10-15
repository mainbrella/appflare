# Mainbrella on Appflare

Research checked October 15, 2026. The standalone app is in the sibling
`../ai-receptionist` repository; its README covers development and deployment,
and `docs/appflare.md` covers submission.

## Where the PR goes

App additions go to [appflare/catalog](https://github.com/appflare/catalog),
not this manager repository. The entry is
`apps/mainbrella-voice/appflare.jsonc`, plus the regenerated `CODEOWNERS`.
Follow the [catalog contribution guide](https://github.com/appflare/catalog/blob/main/CONTRIBUTING.md)
and [submission guide](https://appflare.dev/catalog/submit/).

The app needs a public source repository, a full 40-character commit pin,
a valid license, and a Wrangler configuration whose resources Appflare can
provision. Catalog commits use Conventional Commits, for example
`feat(apps): add mainbrella voice`. Do not edit the generated catalog index.

## Why extract voice

The full Mainbrella backend depends on its private container runtime, preview
and management Workers, platform authentication, and prepaid billing. The
frontend also targets the hosted Mainbrella API and authentication. This is not
a self-contained install.

Appflare supports entries with several Workers, so Worker count alone is not
the blocker. Its [unsupported Wrangler sections](../packages/schema/src/wrangler-sections.ts)
explicitly reject Containers for catalog apps. Its packer also rejects bindings
to Workers outside the entry. Removing these bindings from the full platform
would remove core functionality.

Mainbrella Voice is a useful independent boundary: one Worker serving the UI and
voice API, one D1 database, per-call SQLite Durable Objects, and Workers AI.
Optional Twilio/Telnyx connections use the installer's own carrier accounts.
There are no service bindings or Mainbrella account, billing, or container
dependencies. The extracted source retains its GPL-3.0-only license.

## Packaging

`../ai-receptionist/appflare.jsonc` is the packaging template. After committing
and publishing the app's source, run `npm run catalog:manifest` there. It writes
an entry pinned to the exact clean commit in
`dist/catalog/apps/mainbrella-voice/appflare.jsonc`. The template's zero SHA is
explicitly a sentinel and must not be submitted.

The entry builds with `npm run build:client`, checks `/health`, generates an admin
password, fills the public origin with `{{appUrl}}`, and exposes optional carrier
settings. It targets Workers Paid with a 30-second CPU budget. Its Access bypasses
leave only health and signed carrier endpoints public if an admin enables Access.

In a catalog checkout with the packer built at its `.appflare-ref`, run:

```sh
pnpm install --frozen-lockfile
pnpm gen-codeowners
pnpm check
pnpm validate mainbrella-voice
pnpm pack-app mainbrella-voice
```

Catalog CI validates, packs, verifies hashes, and tests installation where its
Cloudflare credentials are available. Fork PRs do not receive those credentials;
maintainers may need to run the install check. A clean pack is not a live phone
test. Publish the source and verify a fresh Cloudflare install, browser audio,
and the enabled carrier connections before submitting the listing.

No manager runtime change is required for this app.

## Local verification

The extracted app passed TypeScript checks and the browser workflow at 390×844,
768×1024, 1280×800, and 1440×900. Runtime tests use the real Agent and VoiceClient
with deterministic inference, including browser PCM, signed carrier requests,
limits, transcript deletion, retention, and carrier hangup retry handling.

The catalog's pinned packer (`fdf67cb1c680c11dd2a9c0ddd353d131b2d8c6db`)
packed an isolated clean snapshot with a frozen dependency install and no build
credentials: one 2.42 MiB Worker module, eight assets, and one D1 migration.
Artifact hash and upload checks passed. This was an unsigned local verification,
not publication or a remote Cloudflare/carrier install test.
