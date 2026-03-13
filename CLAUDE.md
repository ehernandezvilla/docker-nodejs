# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development (with auto-reload and debugger on port 9229)
npm run dev

# Run tests
npm test

# Run a single test file
npx jest spec/routes/addItem.spec.js

# Format code
npm run prettier

# Docker Compose (local dev with PostgreSQL)
docker compose up --build

# Build Docker image (dev target)
docker build --target dev -t app-dev .

# Build Docker image (prod target)
docker build --target prod -t app-prod .
```

## Architecture

This is a Node.js todo app with a pluggable persistence layer and a React frontend served as static files.

### Persistence layer switching

`src/persistence/index.js` selects the database adapter at runtime: if the `POSTGRES_HOST` environment variable is set, it loads `postgres.js`; otherwise it falls back to `sqlite.js` (default path `/tmp/todo.db`). Both adapters expose the same interface: `init`, `getItems`, `insertItem`, `updateItem`, `removeItem`.

### Request flow

```
HTTP request → src/index.js (Express) → src/routes/*.js → src/persistence/index.js → DB
```

Static files in `src/static/` are served directly by Express. The React app (`src/static/js/app.js`) communicates with the backend via fetch calls to the `/items` REST API.

### Docker / Compose

- `Dockerfile` has two stages: `dev` (runs `npm run dev`, includes devDependencies) and `prod` (runs `node src/index.js`, production deps only).
- `compose.yaml` starts both the `server` (dev stage) and a PostgreSQL `db` service. The DB password is read from `db/password.txt` via Docker secrets. The `db/` directory is gitignored.
- `docker-node-kubernetes.yaml` defines a single-replica Kubernetes Deployment + NodePort Service (port 30001 → 3000).

### CI/CD

`.github/workflows/main.yml` runs on push to `main`. It builds the dev image (to run tests), then builds and pushes the prod image to Docker Hub as `{DOCKER_USERNAME}/{repo}:latest`. Requires `DOCKER_USERNAME` and `DOCKERHUB_TOKEN` secrets.

### Testing

Tests live in `spec/`. Route tests mock the persistence layer; `spec/persistence/sqlite.spec.js` is an integration test that writes to a real SQLite file and cleans up before each test.

### Code style

Prettier is configured in `package.json`: single quotes, 4-space indentation, trailing commas everywhere, semicolons on.
