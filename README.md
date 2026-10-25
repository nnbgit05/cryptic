# Cryptic

A self-hosted password manager with zero-knowledge authentication. The server never sees your master password: login uses the Secure Remote Password protocol (SRP-6a), and vault contents are encrypted in the browser before they are stored.

## How it works

- **Authentication with SRP-6a.** Implemented from scratch against [RFC 5054](https://datatracker.ietf.org/doc/html/rfc5054), using BigInt modular arithmetic and the 4096-bit group in production (1024-bit in tests). Client and server prove they know the same verifier without the password or an equivalent ever crossing the wire.
- **Short-lived login state in Redis.** Each in-progress SRP handshake is cached in Redis and expires after 10 seconds.
- **Session keys from the handshake.** The shared SRP session key is expanded with HKDF-SHA512 into separate encryption and authentication keys on both client and server.
- **Client-side key hierarchy.** AES-256-GCM key-encryption keys wrap the vault keys and an RSA key pair through the Web Crypto API, with keys kept in IndexedDB, so the server only stores wrapped keys and ciphertext.
- **Persistent storage in PostgreSQL.** Users, keychains and vaults are modelled with Sequelize.

## Tech stack

| Layer | Tools |
|-------|-------|
| Frontend | Vanilla JavaScript, Vite, Web Crypto API, IndexedDB |
| Backend | Node.js, Express, vite-express |
| Data | PostgreSQL (Sequelize), Redis |
| Testing | Vitest, Supertest, RFC 5054 test vectors |

## Project structure

```
src/
  client/        Login, register and vault pages, key handling
  server/        Express app, routes, controllers, Sequelize models
  srp/           SRP-6a implementation and RFC 5054 group parameters
tests/           SRP, router and utility tests
cryptic.sh       Install and start script
```

## Requirements

- [Node.js](https://nodejs.org/) 20 or newer, with npm
- [PostgreSQL](https://www.postgresql.org/) running locally, with a database named `knox`
- [Redis](https://redis.io/) running locally on the default port

## Getting started

```bash
git clone https://github.com/nnbgit05/cryptic.git
cd cryptic
```

Either run the setup script, which installs Node via nvm if needed, installs dependencies, builds the client and offers to start the server:

```bash
bash cryptic.sh
```

Or do it by hand:

```bash
npm install
npm run build
npm start
```

The app runs at [http://localhost:3000](http://localhost:3000). For development with auto-reload, use `npm run dev`.

## Tests

```bash
npm test
```

The SRP tests check the multiplier `k`, scrambling parameter `u` and session key `K` against the official RFC 5054 test vectors.

## Note

Knox was the project's working name, so you will still see it in the setup script, the package name and the database name.
