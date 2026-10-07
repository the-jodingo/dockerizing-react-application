[![Docker](https://img.shields.io/badge/Docker-multi--stage-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/)
[![React](https://img.shields.io/badge/React-app-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![Node.js](https://img.shields.io/badge/Node.js-20-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

# Dockerizing a React Application

A reference for containerising a React app: a development image with hot reload,
and a minimal multi-stage production image serving static assets.

## Table of contents

- [Requirements](#requirements)
- [Development image](#development-image)
- [Production image](#production-image)
- [Example Dockerfiles](#example-dockerfiles)
- [Key points](#key-points)
- [License](#license)

## Requirements

| Tool | Version |
|---|---|
| Docker | any recent |
| Node.js | 20 (for local work outside Docker) |

## Development image

Mounts the source and runs the dev server with hot reload:

```bash
docker build -f Dockerfile.dev -t react-app:dev .
docker run -it --rm -p 3000:3000 \
  -v "$(pwd)":/app -v /app/node_modules react-app:dev
```

The second `-v` is deliberate: it keeps the container's `node_modules` from
being shadowed by the bind mount.

## Production image

Compile the bundle, then serve it from a minimal web server:

```bash
docker build -t react-app:prod .
docker run -it --rm -p 8080:80 react-app:prod
```

Open <http://localhost:8080>.

## Example Dockerfiles

**`Dockerfile.dev`**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npm", "start"]
```

**`Dockerfile`** (multi-stage, production)
```dockerfile
# --- build stage ---
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# --- serve stage ---
FROM nginx:1.27-alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Key points

- **Multi-stage builds** keep the runtime image small — build tooling never
  ships to production.
- **`.dockerignore` must include** `node_modules`, `.git`, and `build`/`dist`,
  or you copy a huge context and can shadow the container's own dependencies.
- **Never bake secrets** into an image. Pass them at runtime via environment
  variables or a secrets manager.
- **Pin base image versions** (`node:20-alpine`) rather than using `latest`, so
  builds are reproducible.
- **Use `npm ci`**, not `npm install`, in builds — it honours the lockfile and
  is faster and deterministic.
- **Run as non-root** in production where the base image allows it.

## License

[MIT](LICENSE) © Joash Odingo
