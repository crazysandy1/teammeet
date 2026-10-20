# Teammeet

A team video-conferencing platform in the style of Google Meet. Create a meeting, share the code, and talk, chat and share your screen in the browser. Built with WebRTC for media, Socket.IO for signalling, React for the UI and PostgreSQL for data.

> **Status:** early prototype. Core flows are implemented but untested at scale and without automated tests.

## Features

- Sign up, log in, token refresh (JWT access and refresh tokens).
- Create a meeting and join by meeting code.
- Peer-to-peer video and audio using WebRTC (offer, answer and ICE exchange over Socket.IO).
- In-meeting chat with history.
- Mute and camera toggles, raise and lower hand, screen sharing.
- Host controls: mute or remove a participant, end the meeting.
- Profile with avatar upload (Cloudinary) and password-reset email (Nodemailer).

## Architecture

```
React app  --REST-->  Express API  --->  PostgreSQL (users, meetings, participants, chat_messages)
    |                      |
    +----Socket.IO---------+   signalling: join, offer, answer, ice-candidate, chat, hand, share
    |
    +==== WebRTC (browser to browser media) ==== STUN / optional TURN
```

## Tech stack

- **Frontend:** React 18 (Create React App), React Router 6, socket.io-client, styled-components, Framer Motion.
- **Backend:** Node.js, Express, Socket.IO, `pg`, JWT, bcryptjs, Helmet, rate limiting, express-validator.
- **Services:** PostgreSQL (for example Neon), Cloudinary, an SMTP server, STUN/TURN servers.

## Project layout

```
backend/    Express API, Socket.IO handlers, DB setup (tables created on start)
frontend/   React app: pages (Home, Login, Signup, Dashboard, Meeting, Profile), WebRTC hook
```

## Getting started

Requirements: Node.js 18+ and a PostgreSQL database.

```bash
git clone https://github.com/crazysandy1/teammeet.git
cd teammeet
npm run install-all
cp backend/env.example backend/.env     # fill in the values
npm run dev                              # API :5000 and client :3000
```

### Environment variables (`backend/.env`)

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET`, `JWT_REFRESH_SECRET` | Token signing secrets. Use long random values. |
| `JWT_EXPIRES_IN`, `JWT_REFRESH_EXPIRES_IN` | Token lifetimes (15m and 7d by default) |
| `PORT`, `NODE_ENV`, `CLIENT_URL` | Server settings and allowed client origin |
| `CLOUDINARY_*` | Avatar uploads |
| `EMAIL_HOST`, `EMAIL_PORT`, `EMAIL_USER`, `EMAIL_PASS` | Password-reset email |
| `STUN_SERVER`, `TURN_SERVER`, `TURN_USERNAME`, `TURN_PASSWORD` | WebRTC connectivity |

## API overview

| Area | Endpoints |
|---|---|
| Auth | `POST /signup`, `/login`, `/refresh-token`, `GET /verify` |
| Meetings | `POST /create`, `POST /join`, `GET /:code`, `GET /`, `POST /:id/end`, `GET /:id/participants` |
| Users | `GET /me`, `PUT /update`, `PUT /avatar`, `DELETE /delete` |

Realtime events: `join-meeting`, `offer`, `answer`, `ice-candidate`, `send-message`, `get-chat-history`, `toggle-mute`, `toggle-video`, `raise-hand`, `lower-hand`, `start-screen-share`, `stop-screen-share`, `mute-participant`, `remove-participant`.

## Limitations and roadmap

- Peer-to-peer mesh works for small groups only. Larger meetings need an SFU (for example LiveKit or mediasoup).
- A TURN server is needed for users behind strict NATs.
- Add automated tests, meeting recording, waiting room and end-to-end checks.

## License

MIT
