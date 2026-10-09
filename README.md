# Vchat

Vchat is a mobile-first PWA starter for a messaging app, using the blue accent `#165DFF`, dark/light themes, unique `@username` profiles, PostgreSQL, JWT sessions, and Socket.IO real-time messaging.

## Included in this project

- Responsive PWA shell with Chats / Status / Calls navigation, search, theme toggle, chat list, conversation view, message composer, profile/settings panels.
- Signup/login by phone or email + password, username uniqueness, profile endpoint.
- OTP request/verification API. In development only, `OTP_DEV_MODE=true` logs the OTP to the server console and includes it in the API response. **Turn this off in production and connect an SMS/email provider.**
- PostgreSQL schema for users, sessions, contacts, conversations, members, messages, blocks, statuses, calls, and scheduled messages.
- Socket.IO authentication, conversation rooms, live message delivery, typing indicators, read receipts, and online presence.
- PWA manifest and service worker for installability and basic app-shell caching.
- Safe password hashing, JWT auth, Helmet headers, CORS origin restriction, parameterized SQL, and message length limits.

## Important scope and safety notes

This is a runnable foundation, not a claim that every advanced feature in the supplied specification is already production-complete. The interface exposes core messaging flows; several advanced functions (actual voice/video transport, end-to-end encryption, push notifications, cloud backups, media uploads, native biometric auth, contact sync, status expiration jobs, channels/bots, polls, scheduled delivery and full group-admin UI) require further implementation and service credentials. They are represented in the database schema or documented extension points where appropriate.

Do not advertise end-to-end encryption until a reviewed protocol and audited client implementation are integrated. TLS alone is not E2EE. Do not deploy development OTP mode. Avoid storing passwords, OTPs, private keys, or access tokens in plaintext. Before public launch, add rate limits, abuse reporting, account deletion workflows, email/SMS provider integration, backups, monitoring, security review, and privacy/legal pages.

## Requirements

- Node.js 20+
- Docker (recommended) or PostgreSQL 15+
- npm

## Run locally

1. Copy `.env.example` to `.env` and replace `JWT_SECRET` with a random secret.
2. Start PostgreSQL:

   ```bash
   docker compose up -d db
   ```

3. Install dependencies and create tables:

   ```bash
   npm install
   npm run db:init
   npm run dev
   ```

4. Open `http://localhost:3000`.

For Termux, install Node.js and PostgreSQL or use a remote PostgreSQL/VPS. Docker usually is not available in ordinary Termux setups. A VPS is better for a persistent public service.

## OTP development flow

- Call `POST /api/auth/request-otp` with `{ "identifier": "your phone or email" }`.
- With `OTP_DEV_MODE=true`, the API response and server console show the OTP for local testing.
- Call `POST /api/auth/verify-otp` with `{ "identifier": "...", "otp": "123456" }`.
- For production, set `OTP_DEV_MODE=false` and implement a trusted SMS/email delivery provider before enabling this flow.

## API overview

- `POST /api/auth/register` — `{ phone?, email?, username, password, name }`
- `POST /api/auth/login` — `{ identifier, password }`
- `POST /api/auth/request-otp` — request development OTP
- `POST /api/auth/verify-otp` — verify OTP
- `GET /api/me` — current profile
- `PATCH /api/me` — update name, username, bio, birthday, avatar URL
- `GET /api/conversations` — list conversations and last message
- `POST /api/conversations` — create direct conversation with `{ username }`
- `GET /api/conversations/:id/messages?before=<message-id>` — fetch messages
- `POST /api/conversations/:id/messages` — send a message
- `PATCH /api/messages/:id` — edit own message
- `DELETE /api/messages/:id` — delete own message for everyone
- `GET /api/contacts`, `POST /api/contacts`
- `POST /api/blocks/:userId`, `DELETE /api/blocks/:userId`

Socket events: `conversation:join`, `message:send`, `message:new`, `typing:start`, `typing:stop`, `message:read`, `presence:update`.

## Deploy

Use a managed PostgreSQL database or secure VPS PostgreSQL, set environment variables, run `npm ci --omit=dev`, then `npm start` behind a reverse proxy with HTTPS. Configure `PUBLIC_ORIGIN` to the exact public origin. Use a process manager and regular encrypted backups. Never commit `.env`.
