# Go Boilerplate (backend)

A focused Go backend repository implementing a production-ready REST service with clean architecture. This repo contains the backend code. It uses modern Go libraries and conventions and is intended as a starting point for building a scalable Go service.

Key libraries and tools used
- `github.com/labstack/echo/v4` — HTTP framework
- `github.com/jackc/pgx/v5` — PostgreSQL driver / connection pooling
- `github.com/hibiken/asynq` — Redis-based background job processing
- `github.com/rs/zerolog` — Structured logging
- `github.com/newrelic/go-agent/v3` — New Relic integrations (optional)
- `github.com/resend/resend-go` — Transactional email integration
- `github.com/clerk/clerk-sdk-go` — Authentication SDK (used where appropriate)
- `github.com/jackc/tern` — Database migrations (Taskfile integrates this)
- Task runner: `Taskfile.yml` (uses `task`)

Repository layout

```learn_go_backend/README.md#L1-40
go-starter/
├── cmd/                   # CLI / main entrypoint(s)
│   └── go-boilerplate/    # main application entry (go run ./cmd/go-boilerplate)
├── internal/              # application packages (config, server, handlers, service, repo, etc.)
├── static/                # static assets (if used)
├── templates/             # HTML/email templates
├── Taskfile.yml           # development tasks (run, migrations, tidy, ...)
├── go.mod
└── README.md
```

Quick start

Prerequisites
- Go 1.24+
- PostgreSQL (the DB)
- Redis (for caching / background workers)
- `task` (recommended) — used to run tasks in `Taskfile.yml`
- `tern` CLI (for migrations) — used by `task migrations:new` / `task migrations:up`

Install and run locally

```learn_go_backend/README.md#L41-70
# 1. Clone the repo
git clone <your-repo-url>
cd learn_go_backend

# 2. Download Go dependencies
go mod download
```

Configuration / environment variables

This service loads configuration from environment variables prefixed with `BOILERPLATE_`. The loader lowercases and maps nested fields using dots/underscores. Example environment keys you will commonly set:

- `BOILERPLATE_PRIMARY_ENV` — runtime environment (e.g. `local`, `development`, `production`)
- `BOILERPLATE_SERVER_PORT` — HTTP server port (string)
- `BOILERPLATE_SERVER_READ_TIMEOUT` — read timeout (int seconds)
- `BOILERPLATE_SERVER_WRITE_TIMEOUT` — write timeout (int seconds)
- `BOILERPLATE_DATABASE_HOST`
- `BOILERPLATE_DATABASE_PORT`
- `BOILERPLATE_DATABASE_USER`
- `BOILERPLATE_DATABASE_PASSWORD`
- `BOILERPLATE_DATABASE_NAME`
- `BOILERPLATE_DATABASE_SSLMODE`
- `BOILERPLATE_DATABASE_MAX_OPEN_CONNS`
- `BOILERPLATE_REDIS_ADDRESS`
- `BOILERPLATE_INTEGRATION_RESEND_API_KEY`
- `BOILERPLATE_AUTH_SECRET_KEY`
- `BOILERPLATE_OBSERVABILITY_NEW_RELIC_LICENSE_KEY`

You can export these variables, or place them in an env file and load them in your shell. The code uses `koanf` + env provider and expects full variable names as shown.

Example `.env` snippet

```learn_go_backend/README.md#L71-100
BOILERPLATE_PRIMARY_ENV=local
BOILERPLATE_SERVER_PORT=8080
BOILERPLATE_DATABASE_HOST=127.0.0.1
BOILERPLATE_DATABASE_PORT=5432
BOILERPLATE_DATABASE_USER=myuser
BOILERPLATE_DATABASE_PASSWORD=mypassword
BOILERPLATE_DATABASE_NAME=mydb
BOILERPLATE_DB_DSN=postgres://myuser:mypassword@127.0.0.1:5432/mydb?sslmode=disable
BOILERPLATE_REDIS_ADDRESS=127.0.0.1:6379
BOILERPLATE_INTEGRATION_RESEND_API_KEY=your_resend_key
BOILERPLATE_AUTH_SECRET_KEY=supersecret
BOILERPLATE_OBSERVABILITY_NEW_RELIC_LICENSE_KEY=
```

Database migrations

`Taskfile.yml` exposes migration tasks which use `tern`. The Taskfile expects the environment variable `BOILERPLATE_DB_DSN` for migration commands.

```learn_go_backend/README.md#L101-130
# Apply all up migrations (Taskfile will prompt for confirmation)
export BOILERPLATE_DB_DSN="postgres://user:pass@host:5432/db?sslmode=disable"
task migrations:up

# Create a new migration (uses tern)
task migrations:new name=add_users_table
```

Run the application

```learn_go_backend/README.md#L131-160
# Using the Taskfile
task run

# Or run directly with go
go run ./cmd/go-boilerplate
```

Notes about running
- The server entrypoint is in `cmd/go-boilerplate/main.go`.
- The application will automatically run DB migrations on startup when `Primary.Env` is not `local` (see config / code). For local development you can run migrations manually with the Taskfile.
- The code gracefully handles shutdown signals and uses a default context timeout of 30s.

Development tasks

```learn_go_backend/README.md#L161-190
# View available tasks
task --list-all

# Format and tidy
task tidy

# Run tests
go test ./...

# Integration tests (Docker required)
go test -tags=integration ./...
```

Architecture overview

- `cmd/` — application entrypoint(s)
- `internal/config` — configuration and validation
- `internal/server` — server bootstrap, HTTP server setup
- `internal/router` — HTTP routes and middleware
- `internal/handlers` — HTTP handlers
- `internal/service` — business logic
- `internal/repository` — data access (Postgres/Redis)
- `internal/logger` — logging and observability integration
- `internal/database` — migrations and DB initialization
- `internal/middleware` — HTTP middleware (auth, rate limit, CORS)

Observability and monitoring
- The repository includes New Relic integrations. Observability settings are optional and have sensible defaults. Configure New Relic with `BOILERPLATE_OBSERVABILITY_NEW_RELIC_LICENSE_KEY` and other `BOILERPLATE_OBSERVABILITY_*` variables if you want APM and log forwarding.

Testing
- Unit tests: `go test ./...`
- Integration tests: `go test -tags=integration ./...` (Docker required)
- Use the Taskfile `task test` if present for convenience.

Contributing
1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/awesome-thing`
3. Run tests and linters locally.
4. Open a PR with a clear description and tests where applicable.

License
- This project is provided under the MIT License. See the `LICENSE` file for details.
