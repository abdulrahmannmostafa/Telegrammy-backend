# Telegrammy Backend

A production-oriented Node.js backend for a real-time messaging platform with support for authentication, chats, groups, channels, media uploads, stories, notifications, search, and voice calling.

## Table of Contents

- [Overview](#overview)
- [Core Features](#core-features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Architecture at a Glance](#architecture-at-a-glance)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Variables](#environment-variables)
  - [Run the Application](#run-the-application)
- [API Documentation](#api-documentation)
- [Key HTTP Modules](#key-http-modules)
- [Real-time Events (Socket.IO)](#real-time-events-socketio)
- [Testing](#testing)
- [Linting and Formatting](#linting-and-formatting)
- [Docker](#docker)
- [CI/CD](#cicd)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Overview

Telegrammy Backend is the server-side implementation of the Telegrammy application. It exposes REST APIs and real-time Socket.IO events for modern communication workflows:

- account lifecycle (registration, verification, login/logout, recovery)
- one-to-one and group communication
- channel management and participant roles
- media and story handling with AWS S3
- secure session and token-based authentication
- live voice call signaling over WebSocket events
- API documentation through Swagger UI

## Core Features

- **Authentication & Authorization**
  - Email/password login and registration
  - OAuth sign-in (Google, GitHub)
  - Password reset and “logout from all devices”
  - Route protection via authentication middleware

- **Messaging Domain**
  - Chat retrieval and contact discovery
  - Message send/update/delete/seen events
  - Draft syncing and event acknowledgment

- **Groups & Channels**
  - Group administration, member permissions, and moderation controls
  - Channel creation, privacy settings, invite links, and role promotions

- **Media & Stories**
  - Multipart upload support for media, audio, documents, and stickers
  - Story creation and contact-based visibility flow

- **Calls & Presence**
  - WebRTC call signaling (offer/answer/ICE events)
  - Call state updates and history endpoints

- **Operational Capabilities**
  - Redis adapter for scalable Socket.IO pub/sub
  - MongoDB persistence through Mongoose models
  - Firebase integration for push notifications

## Tech Stack

- **Runtime:** Node.js (CommonJS)
- **Framework:** Express.js
- **Database:** MongoDB + Mongoose
- **Realtime:** Socket.IO + Redis adapter
- **Auth:** Passport, JWT, sessions (Mongo-backed)
- **Storage:** AWS S3 (multer + multer-s3)
- **Notifications:** Firebase Admin, SendGrid
- **Validation:** express-validator
- **API docs:** swagger-jsdoc + swagger-ui-express
- **Testing:** Jest, Mocha, Chai, Supertest, Sinon
- **Quality:** ESLint, Prettier, NYC

## Project Structure

```text
src/
  classes/         # AI inference/classification strategy helpers
  config/          # ICE server and socket-related config
  controllers/     # HTTP controller logic by domain
  docs/            # Swagger/OpenAPI source docs and artifacts
  errors/          # Custom errors and error handlers
  eventHandlers/   # Socket.IO event handlers (chat/group/channel/call)
  middlewares/     # Authentication, AWS upload, validation, etc.
  models/          # Mongoose schemas
  routes/          # Express route modules
  services/        # Business/service-layer operations
  tests/           # Unit/integration tests
  utils/           # Cross-cutting utility modules
  expressApp.js    # Express app wiring
  ioApp.js         # Socket.IO initialization
  server.js        # HTTP/HTTPS server + DB bootstrap
  index.js         # Entry point
```

## Architecture at a Glance

1. `src/index.js` boots the server.
2. `src/server.js` initializes HTTP/HTTPS, connects MongoDB, and starts listening.
3. `src/expressApp.js` wires middleware and mounts all REST routes under `/api/v1`.
4. `src/ioApp.js` mounts Socket.IO on the same server and configures Redis adapter.
5. `src/eventHandlers/*` manages live events for chat, groups, channels, and calls.

## Getting Started

### Prerequisites

- Node.js 18+ recommended
- npm
- MongoDB instance
- Redis instance (for Socket.IO adapter)
- AWS S3 credentials (for uploads)

### Installation

```bash
npm install
```

### Environment Variables

Create a `.env` file in the repository root and configure the variables used by the application.

#### Server & Core

- `NODE_ENV`
- `HOST_NAME`
- `PORT`
- `DB_HOST`
- `REDIS_URL`
- `SERVER_WORKER_ID`

#### Authentication & Session

- `JWT_SECRET`
- `JWT_ACCESS_EXPIRES_IN_HOURS`
- `JWT_REFRESH_EXPIRES_IN_DAYS`
- `COOKIE_ACCESS_NAME`
- `COOKIE_REFRESH_NAME`
- `COOKIE_ACCESS_EXPIRES_IN_HOURS`
- `COOKIE_REFRESH_EXPIRES_IN_DAYS`
- `FRONTEND_LOGIN_CALLBACK`
- `SET_PASSWORD_URL`
- `RESET_PASSWORD_TOKEN_DURATION`
- `RESEND_PASSWORD_TOKEN_COOLDOWN`

#### OAuth Providers

- `GOOGLE_CLIENT_ID`
- `GOOGLE_CLIENT_SECRET`
- `GOOGLE_CALLBACK_URL`
- `GITHUB_CLIENT_ID`
- `GITHUB_CLIENT_SECRET`
- `GITHUB_CALLBACK_URL`

#### Email & Notifications

- `APP_EMAIL`
- `SENDGRID_API_KEY`
- `SNDGRID_TEMPLATEID_REGESTRATION_EMAIL`
- `SNDGRID_TEMPLATEID_UPDATING_EMAIL`

#### Firebase

- `FIREBASE_TYPE`
- `FIREBASE_PROJECT_ID`
- `FIREBASE_PRIVATE_KEY_ID`
- `FIREBASE_PRIVATE_KEY`
- `FIREBASE_CLIENT_EMAIL`
- `FIREBASE_CLIENT_ID`
- `FIREBASE_AUTH_URI`
- `FIREBASE_TOKEN_URI`
- `FIREBASE_AUTH_PROVIDER`
- `FIREBSAE_CLIENT_CERT`
- `FIREBASE_UNIVERSE_DOMAIN`

#### AWS & Media

- `AWS_REGION`
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_BUCKET_NAME`

#### Security & RTC

- `CAPTCHA_SECRET`
- `STUN_SERVER_URL`
- `TURN_SERVER_URL`
- `TURN_SERVER_USERNAME`
- `TURN_SERVER_CREDENTIAL`

### Run the Application

```bash
# development (nodemon)
npm run start

# production mode (nodemon + NODE_ENV=production)
npm run start:prod
```

Server entrypoint: `src/index.js`

## API Documentation

Swagger UI is exposed at:

```text
/api-docs
```

The OpenAPI spec is generated from `src/docs/*.js` via `swaggerConfig.js`.

To export docs as JSON:

```bash
node saveApiDocumentationFile.js
```

## Key HTTP Modules

All endpoints are mounted under `/api/v1`:

- `/auth` - login, logout, registration, OAuth, recovery
- `/user` - main user APIs
- `/user/profile` - profile settings and identity updates
- `/user/stories` - story creation/read/delete flows
- `/privacy/settings` - privacy and blocking controls
- `/messaging/upload` - media/audio/document/sticker uploads
- `/chats` - chat retrieval and contact fetch
- `/groups` - group control and permissions
- `/channels` - channel management and invites
- `/search` - scoped and global search
- `/call` - call list, chat call history, join operations
- `/admins` - admin-level user/group moderation
- `/notification` - mute/unmute APIs
- `/ice-servers` - STUN/TURN configuration endpoint

## Real-time Events (Socket.IO)

Primary runtime events include:

- **Messaging:** `message:send`, `message:update`, `message:delete`, `message:seen`
- **Draft/Typing:** `draft`, `typing`, `event:ack`
- **Calls:** `call:createCall`, `call:offer`, `call:answer`, `call:end`, `call:reject`, `call:addIce`
- **Groups:** `creatingGroup`, `addingGroupMember`, `addingGroupMemberV2`, `leavingGroup`, `removingGroup`
- **Channels:** `addingChannelSubscriper`, `promoteSubscriper`, `demoteAdmin`, `removingChannel`

## Testing

```bash
# jest
npm test

# mocha suite
npm run test-mocha

# coverage (nyc + mocha)
npm run coverage
```

Test files are located under `src/tests`.

## Linting and Formatting

```bash
npm run lint
```

ESLint and Prettier configuration are present in:

- `.eslintrc.json`
- `.prettierrc`

## Docker

Build and run locally:

```bash
docker build -t telegrammy-backend .
docker run --env-file .env -p 8080:8080 telegrammy-backend
```

The `Dockerfile` uses:

- `node:18-slim`
- non-root runtime user (`telegrammy`)
- production dependency install via `npm ci --only=production`

## CI/CD

`Jenkinsfile` currently defines:

- dependency installation
- test execution (`npm test`)
- docker image build
- conditional image push and SonarCloud scan
- manifest update pipeline for main branch deployments

## Troubleshooting

- **Mongo connection fails:** verify `DB_HOST` and database reachability.
- **Socket issues across instances:** ensure `REDIS_URL` is valid and reachable.
- **Uploads fail:** confirm AWS credentials, bucket name, and object permissions.
- **OAuth callback errors:** verify provider callback URLs match configured app settings.
- **Missing notifications:** validate Firebase service account environment variables.

## Contributing

1. Create a feature branch.
2. Keep changes scoped and well-tested.
3. Run lint and tests before opening a PR.
4. Update API docs if endpoint behavior changes.
