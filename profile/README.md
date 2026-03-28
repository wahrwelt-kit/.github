# go-kit

Opinionated Go toolkit - a collection of focused libraries for building production-ready backends.

Each kit is an independent module with its own versioning, CI, and minimal dependency surface.

## Kits

| Kit                                                         | Description                                                | Version                                                                                                                                       |
| ----------------------------------------------------------- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| [go-logkit](https://github.com/wahrwelt-kit/go-logkit)     | Structured logging (zerolog) with context, slog bridge, noop     | [![Go Reference](https://pkg.go.dev/badge/github.com/wahrwelt-kit/go-logkit.svg)](https://pkg.go.dev/github.com/wahrwelt-kit/go-logkit)     |
| [go-httpkit](https://github.com/wahrwelt-kit/go-httpkit)   | HTTP middleware (i18n, metrics, timeout, JWT, IP), error handling | [![Go Reference](https://pkg.go.dev/badge/github.com/wahrwelt-kit/go-httpkit.svg)](https://pkg.go.dev/github.com/wahrwelt-kit/go-httpkit)   |
| [go-pgkit](https://github.com/wahrwelt-kit/go-pgkit)       | PostgreSQL pool, migrations (goose/migrate), error helpers (pgx)  | [![Go Reference](https://pkg.go.dev/badge/github.com/wahrwelt-kit/go-pgkit.svg)](https://pkg.go.dev/github.com/wahrwelt-kit/go-pgkit)       |
| [go-jwtkit](https://github.com/wahrwelt-kit/go-jwtkit)     | JWT auth — symmetric & asymmetric, middleware, revocation         | [![Go Reference](https://pkg.go.dev/badge/github.com/wahrwelt-kit/go-jwtkit.svg)](https://pkg.go.dev/github.com/wahrwelt-kit/go-jwtkit)     |
| [go-cachekit](https://github.com/wahrwelt-kit/go-cachekit) | Redis cache + in-memory LRFU, singleflight, KV store, Pub/Sub    | [![Go Reference](https://pkg.go.dev/badge/github.com/wahrwelt-kit/go-cachekit.svg)](https://pkg.go.dev/github.com/wahrwelt-kit/go-cachekit) |
| [go-wskit](https://github.com/wahrwelt-kit/go-wskit)       | WebSocket + SSE hub with Redis Pub/Sub scaling                    | [![Go Reference](https://pkg.go.dev/badge/github.com/wahrwelt-kit/go-wskit.svg)](https://pkg.go.dev/github.com/wahrwelt-kit/go-wskit)       |

## Dependency Graph

```mermaid
graph LR
    httpkit -->  logkit
    jwtkit -->  logkit
    wskit -->  cachekit
    pgkit
    cachekit
    logkit
```

## Quick Start

```bash
go get github.com/wahrwelt-kit/go-logkit@latest
go get github.com/wahrwelt-kit/go-httpkit@latest
go get github.com/wahrwelt-kit/go-pgkit@latest
go get github.com/wahrwelt-kit/go-jwtkit@latest
go get github.com/wahrwelt-kit/go-cachekit@latest
go get github.com/wahrwelt-kit/go-wskit@latest
```

Minimal chi server with logging, JWT auth, PostgreSQL and Redis cache:

```go
r := chi.NewRouter()
r.Use(middleware.RequestID())
r.Use(middleware.Logger(log, nil))
r.Use(middleware.Recoverer(log))
r.Use(middleware.Metrics(nil, httputil.ChiPathFromRequest))
r.Use(middleware.Timeout(10 * time.Second))
r.Use(middleware.I18n(bundle, middleware.WithLanguageQueryParam("lang")))

r.Get("/health", httputil.HealthHandler(nil))
r.Get("/metrics", promhttp.Handler().ServeHTTP)

r.Group(func(r chi.Router) {
    r.Use(jwtkit.JWTAuth(jwtSvc, jwtkit.WithLogger(log)))
    r.Get("/users/{id}", getUserHandler(pool, cache, lrfu))
    r.Get("/users/{id}/export", exportUserHandler(pool, cache))
})
```

Full working examples in [`examples/`](examples/).

## Examples

| Example                               | Router | Kits used                                |
| ------------------------------------- | ------ | ---------------------------------------- |
| [chi-rest](examples/chi-rest)         | chi    | logkit, httpkit, pgkit, jwtkit, cachekit |
| [chi-realtime](examples/chi-realtime) | chi    | logkit, httpkit, wskit, cachekit         |
| [gin-rest](examples/gin-rest)         | gin    | logkit, pgkit, jwtkit, cachekit          |
| [gin-realtime](examples/gin-realtime) | gin    | logkit, wskit, cachekit                  |

## Benchmarks

Aggregated benchmark results from all kits are available in [`benchmarks/`](benchmarks/).

Run locally:

```bash
./benchmarks/run.sh
```

## Design Principles

- **Independent modules** - each kit has its own `go.mod`, semver, and CI pipeline
- **Functional options** - consistent configuration pattern across all kits
- **Consumer-side interfaces** - interfaces defined where they are used, not where implemented
- **Zero framework lock-in** - middleware works with `http.Handler`; chi is preferred, not required
- **Minimal dependencies** - each kit pulls only what it needs
