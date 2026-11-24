# CLAUDE.md

Mini-CDN: a CDN simulation in Go (module `mini-cdn`, Go 1.22, standard library only). Client -> router -> edge (disk cache) -> origin.

## Commands

```bash
go build ./...
go vet ./...
go test ./...
gofmt -l .                      # must print nothing
docker compose up --build       # origin + 3 edges + router + Prometheus + Grafana
```

Run a single service locally: `go run ./origin`, `go run ./edge`, `go run ./router` (config via env vars, see `.env.example`).

## Layout

- `router/`: reverse proxy; consistent-hash (`internal/hashring`) or round-robin edge selection, 2 s health probes, load shedding (`MAX_INFLIGHT`), GET/HEAD failover, fans out `/purge` and `/cache/*`.
- `edge/`: disk cache with TTL + LRU, stale-on-origin-error, streams origin body to client while caching.
- `origin/`: read-only static file server (`origin/files`).
- `internal/metrics`: small hand-rolled Prometheus registry. `internal/safepath`: path cleaning against traversal.
- `deploy/`: Prometheus and Grafana provisioning. `Dockerfile` builds one service via `--build-arg SERVICE=<dir>`.

## Conventions and gotchas

- No third-party dependencies; keep it that way unless asked.
- Config is env-only, read via small `env`/`intEnv` helpers in each `main.go`.
- Response headers (`X-Cache`, `X-Edge-Selected`, `X-Load-Shed`, ...) are part of the documented behavior; README demos depend on them.
- Cache paths must go through `internal/safepath`.
- Always run `gofmt`, `go vet` and `go test ./...` before committing; CI runs the same.
