[![Docker](https://img.shields.io/badge/Docker-multi--stage-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/)
[![React](https://img.shields.io/badge/React-app-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

# Dockerizing a React Application

A reference for containerising a React app: a development setup with hot reload,
and a production image served as static assets.

## Development image

Mounts the source and runs the dev server with hot reload:

```bash
docker build -f Dockerfile.dev -t react-app:dev .
docker run -it --rm -p 3000:3000 -v "$(pwd)":/app -v /app/node_modules react-app:dev
```

## Production image

Multi-stage build: compile the bundle, then serve it from a minimal web server.

```bash
docker build -t react-app:prod .
docker run -it --rm -p 8080:80 react-app:prod
# open http://localhost:8080
```

## Key points

- **Multi-stage builds** keep the runtime image small — build dependencies never
  ship to production.
- **`.dockerignore`** must include `node_modules`, `.git`, and build output,
  otherwise you copy a huge context and can shadow the container's own deps.
- **Never bake secrets** into an image; pass them at runtime via environment
  variables or a secrets manager.
- Pin the base image version rather than using `node:latest`.

## License

MIT
