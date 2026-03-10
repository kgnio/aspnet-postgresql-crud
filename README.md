# ASP.NET Core + PostgreSQL CRUD (Dockerized)

This project now supports one-command local startup with Docker Compose.

```

<img width="1808" height="814" alt="resim" src="https://github.com/user-attachments/assets/d08d899c-908a-4f20-bc2a-1d95f85e3066" />

```

## Why these Docker files exist

- `docker-compose.yml`: Starts the API and PostgreSQL together, wires networking, sets env vars, and persists DB data in a named volume.
- `backend/Api/Dockerfile`: Builds and runs the ASP.NET Core API in a container using a multi-stage build.
- `backend/Api/.dockerignore`: Keeps Docker build context small by excluding `bin/`, `obj/`, and editor/system files.
- `.env.example`: Documents PostgreSQL environment variables used by Compose.

## Prerequisites

- Docker Desktop (or Docker Engine + Docker Compose plugin)

## Quick Start (Single Command)

1. Optional: create a `.env` file from `.env.example` if you want custom DB values.
2. Run:

```bash
docker compose up --build
```

Swagger URL:

- `http://localhost:5065/swagger`

## Stop the stack

```bash
docker compose down
```

To also remove DB data volume:

```bash
docker compose down -v
```

## View logs

All services:

```bash
docker compose logs -f
```

API only:

```bash
docker compose logs -f api
```

PostgreSQL only:

```bash
docker compose logs -f db
```

## Connect to PostgreSQL

From host tools (DBeaver, pgAdmin, psql):

- Host: `localhost`
- Port: `5432`
- Database: `AspNetCrudDb` (or your `.env` value)
- Username: `postgres` (or your `.env` value)
- Password: `postgres123` (or your `.env` value)

From inside container:

```bash
docker compose exec db psql -U postgres -d AspNetCrudDb
```

## EF Core migrations

Current setup applies existing migrations automatically on API startup (`Database.Migrate()` with retry), which is convenient for local Docker development.

Useful commands from `backend/Api`:

Create a new migration:

```bash
dotnet ef migrations add <MigrationName>
```

Apply migrations locally (non-Docker run):

```bash
dotnet ef database update
```

In Docker Compose, migrations are applied automatically when `api` starts.

## Docker networking in this project

- Compose creates a private network for services.
- `api` reaches PostgreSQL using the service name `db` as hostname.
- Connection string is injected with:
  - `ConnectionStrings__DefaultConnection=Host=db;Port=5432;...`
- `depends_on` + PostgreSQL healthcheck help startup ordering.
- API also retries migrations at startup in case DB is still initializing.

