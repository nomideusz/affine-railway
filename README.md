# Deploy and Host AFFiNE on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/affine-production?utm_medium=integration&utm_source=button&utm_campaign=affine-production)

[AFFiNE](https://affine.pro/) is the open-source Notion and Miro alternative: docs, an infinite edgeless whiteboard and databases in one workspace, with real-time collaboration, sharing, version history and desktop and mobile apps that sync to your own server. This template runs the official AFFiNE server with Postgres (with pgvector) and Redis, and keeps uploads and the server's signing key on a volume.

## About Hosting AFFiNE

The stack is three services: AFFiNE, Postgres and Redis.

- **Upstream's own image, pinned.** AFFiNE runs from the official `ghcr.io/toeverything/affine` image at a fixed release tag (0.27.4), not the moving `stable` tag, so a redeploy never surprises you with an untested version.
- **Migrations on every boot.** The start command runs AFFiNE's own self-host pre-deploy script (Prisma migrations plus data migrations) before starting the server, and Railway's health check on `/info` only switches traffic once they finish. Upgrading is bumping the image tag.
- **A signing key that is yours.** On first boot the pre-deploy script generates the server's private key into `/root/.affine/config/private.key` on the volume. It signs sessions and tokens, is unique to your deployment and survives redeploys.
- **Uploads on a volume.** Images, attachments and other blobs are stored under `/root/.affine/storage` on the same persistent volume.
- **Postgres with pgvector.** AFFiNE's AI features store embeddings with pgvector, so the database runs the official `pgvector/pgvector` Postgres 17 image. Redis keeps an append-only file on its own volume.

## Common Use Cases

- A team wiki and docs space that replaces Notion, on your own infrastructure
- Brainstorming, diagrams and planning on an infinite whiteboard next to the docs
- Lightweight project tracking with database views (table and kanban) inside pages
- A private sync server for the AFFiNE desktop and mobile apps

## Dependencies for AFFiNE Hosting

- Postgres 17 with pgvector (included, private network only)
- Redis 8 (included, private network only)

### Deployment Dependencies

- [AFFiNE self-hosting documentation](https://docs.affine.pro/self-host-affine)
- [AFFiNE on GitHub](https://github.com/toeverything/AFFiNE)
- [Template source on GitHub](https://github.com/nomideusz/affine-railway)

### Implementation Details

**Create the admin account first.** Open the AFFiNE service's Railway domain as soon as the deploy is green: a new server sends you to the setup page, and the first account created there becomes the instance admin. Until then anyone who finds the URL could claim it. After that, manage users and sign-up settings at `/admin`.

**Desktop and mobile apps.** In the AFFiNE app choose "Add a server" and enter your Railway URL to sync to it.

**Custom domain.** After adding one, set `AFFINE_SERVER_EXTERNAL_URL` to `https://your-domain` so invite links and app sign-in use it.

**Email (optional).** Invitations, magic sign-in links and password resets need SMTP: fill in the `MAILER_*` variables. Railway only allows outbound SMTP on the Pro plan; without mail, the admin can create accounts and reset passwords from `/admin`.

**Back up the volume.** `/root/.affine` holds every uploaded file and `config/private.key`. Losing the key logs everyone out; losing `storage` loses attachments.

**Full-text search indexer.** AFFiNE's optional server-side indexer needs a separate Manticore or Elasticsearch service, so it is off (`AFFINE_INDEXER_ENABLED=false`), as in upstream's own self-host compose. Search inside the apps still works locally.

**Resources.** In testing the AFFiNE server idled at 200–300 MB of RAM and the whole stack at about 450 MB, so it runs on the Hobby plan.

## Why Deploy AFFiNE on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying AFFiNE on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
