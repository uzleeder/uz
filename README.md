# UzLeeder · Max’s Code

A single Node.js application containing the UzLeeder homepage, a public coding journal and a password-protected content studio. No third-party runtime dependencies. No paid service is required by the code.

## Run

Requires Node.js 24 or newer (built-in SQLite).

1. Copy `.env.example` to `.env`.
2. Set a unique `ADMIN_PASSWORD` of at least 16 characters. Keep `.env` private.
3. Set `SITE_URL` to the actual origin, e.g. `http://localhost:3000` locally or `https://uzleeder.uz` in production.
4. Run `npm start` and open `http://localhost:3000`.
5. Sign in at `/admin`. Create content, save drafts, use Preview, then set publication to `published` and save.

`npm run check` checks JavaScript syntax; `npm test` runs real HTTP/database integration tests against an isolated temporary database.

## Routes

- `/`: UzLeeder homepage
- `/maxcode`: Max’s Code overview
- `/maxcode/journal`, `/maxcode/journal/:slug`
- `/maxcode/projects`, `/maxcode/projects/:slug`
- `/maxcode/roadmap`
- `/maxcode/snippets`, `/maxcode/snippets/:slug`
- `/maxcode/about`
- `/admin`: overview, content CRUD, draft previews, media library, settings
- `/sitemap.xml`, `/robots.txt`

## Content management

Journal, projects, milestones, snippets and settings are persisted in `DATA_DIR/uzleeder.sqlite`. First startup creates the schema and empty settings; it creates no fake posts or achievements. Slugs are unique within each content type. Projects have independent publication and development stages. Roadmap progress comes only from records created by the administrator. Feature projects using the editor checkbox.

The editor supports plain Markdown headings and fenced code blocks, with conservative keyword coloring and accessible copy controls. HTML is always escaped. Snippets use the content field as raw code. Article structure is entirely editable. Content has an Uzbek/English language field; translations are separate reviewed records and never auto-published. Search scans published titles, summaries, body and tags using the embedded database; filters use categories and programming-language fields. This is suitable for a personal journal, not a large search index.

Upload PNG, JPEG or WebP images through Media (5 MB maximum), then paste the returned `/media/:id` path into cover or gallery fields. Gallery/resource URLs use one line per item. Related content uses comma-separated numeric IDs shown in the content list; only published related records appear publicly. Media URLs are public: do not upload confidential draft assets. Videos use external links; inline embeds and video uploads are not implemented.

## Production deployment

Use one existing Node-capable host or a single container with a **persistent volume** mounted at `/app/data`. SQLite requires a persistent writable disk and a single application instance. Ephemeral/serverless-only hosting is unsuitable without replacing the storage adapter. Set `SITE_URL=https://uzleeder.uz`, `COOKIE_SECURE=true`, and the password through your host’s secrets manager. Terminate HTTPS at your reverse proxy, forward traffic to port 3000 and keep the application port private. Configure request-size/time limits at the proxy. All internal redirects remain within this app.

Docker build: `docker build -t uzleeder .`. Run with a private environment file and persistent volume: `docker run --env-file .env -p 127.0.0.1:3000:3000 -v uzleeder-data:/app/data uzleeder`.

Back up SQLite using the SQLite online backup API/tool or stop the app and back up the entire data directory including WAL files. Test restoration before relying on backups. Do not copy only the main database file during writes. Protect backups and disk access. Future schema changes must use versioned additive migrations and a verified backup; current initial schema is version 1. Do not delete an existing data volume during upgrades.

The initial repository was empty, so no hosting, DNS, authentication, database or previous brand system could be audited or reused. This app establishes those conventions and does not claim to be deployed at the live domain.

## Security

- Server-side session checks on every admin route and mutation; cryptographically random session and CSRF tokens.
- HttpOnly, SameSite=Strict cookies; Secure when configured for HTTPS. Eight-hour expiry, logout invalidation, every application restart invalidates all existing sessions (including password rotation).
- Login throttling by direct peer IP, deliberately conservative behind a proxy. Restart clears the in-memory rate limiter; enforce durable perimeter throttling for exposed production deployments.
- Constant-time comparison of scrypt password hashes. Passwords are supplied through environment secrets, never stored in the repository.
- SQL parameter binding, safe URL protocols, HTML escaping, upload signatures and size limits, no SVG/HTML uploads.
- CSP, anti-framing headers, private previews marked noindex and no-store, unpublished entries return 404 publicly.
- Single owner account by design. No self-registration, password reset or multi-user roles.

## Optional Telegram

Set `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` on the server, ensure the bot can publish to the channel, and restart. In Journal management expand Telegram on a published entry, review the displayed summary and type `SEND`. Website publication never automatically sends a message. The real result is persisted independently (`pending`, `sent`, `unconfirmed`). An attempted send cannot automatically be retried; inspect Telegram first to avoid duplicates after a timeout. Credentials and publishing permissions have not been verified by this repository. No message is sent during tests.

## Optional subdomain redirect

DNS and hosting providers are unknown. Do not change DNS until the owner approves it. On the existing host, confirm custom-hostname, TLS and redirect support and plan limits. If supported, add `code.uzleeder.uz` using the host’s prescribed DNS target, provision a valid certificate, and configure a host-specific HTTPS **308** redirect to `https://uzleeder.uz/maxcode` preserving the query string. A DNS CNAME alone does not redirect HTTP. Test HTTPS, query parameters, the canonical destination and loop prevention before enabling the permanent redirect. Never create a second content application. The main app’s canonical origin is `SITE_URL`.

## Cost and known limits

No paid SDKs or additional subscriptions are introduced. Actual hosting, disk, bandwidth, domain and backup costs depend on the chosen provider and are not guaranteed to be zero. DNS and deployment were not changed by implementing this repository. External images depend on the configured source; use the media library for assets you control. UI uses mostly English developer terminology with Uzbek main navigation, rather than a full UI translation system. No visitor analytics, video hosting, embedded video player, email subscriptions or automatic translations are claimed.
