# wahrwelt-kit

Focused Go modules for production backend services.

Each kit is versioned and released independently. The [`go-kit`](https://github.com/wahrwelt-kit/go-kit) repository is the composition layer with runnable examples showing how the modules fit together.

## Kits

| Kit | Latest | Description |
| --- | --- | --- |
| [go-logkit](https://github.com/wahrwelt-kit/go-logkit) | `v0.6.0` | Structured logging on top of zerolog: JSON defaults, redaction, async writer, hooks, context fields, `log/slog` bridge |
| [go-httpkit](https://github.com/wahrwelt-kit/go-httpkit) | `v0.7.0` | HTTP helpers for JSON APIs: errors, rendering, decoding, validation, pagination, SSE, health, middleware |
| [go-httpkit/metrics](https://github.com/wahrwelt-kit/go-httpkit/tree/main/metrics) | `v0.7.0` | Prometheus HTTP middleware as a separate module |
| [go-httpkit/localization](https://github.com/wahrwelt-kit/go-httpkit/tree/main/localization) | `v0.7.0` | go-i18n request localization middleware as a separate module |
| [go-pgkit](https://github.com/wahrwelt-kit/go-pgkit) | `v1.4.0` | PostgreSQL pool setup, pgx helpers, transaction helpers, goose and golang-migrate runners |
| [go-jwtkit](https://github.com/wahrwelt-kit/go-jwtkit) | `v0.6.0` | JWT access/refresh pairs, symmetric and asymmetric keys, strict claims, JWK/JWKS helpers, revocation |
| [go-cachekit](https://github.com/wahrwelt-kit/go-cachekit) | `v0.7.0` | Redis JSON cache, SIEVE in-memory cache, TTL single-value cache, key-value and Pub/Sub helpers |
| [go-wskit](https://github.com/wahrwelt-kit/go-wskit) | `v0.4.0` | WebSocket and SSE hub on coder/websocket with optional Redis Pub/Sub fan-out |

## Install

```bash
go get github.com/wahrwelt-kit/go-logkit@v0.6.0
go get github.com/wahrwelt-kit/go-httpkit@v0.7.0
go get github.com/wahrwelt-kit/go-httpkit/metrics@v0.7.0
go get github.com/wahrwelt-kit/go-httpkit/localization@v0.7.0
go get github.com/wahrwelt-kit/go-pgkit@v1.4.0
go get github.com/wahrwelt-kit/go-jwtkit@v0.6.0
go get github.com/wahrwelt-kit/go-cachekit@v0.7.0
go get github.com/wahrwelt-kit/go-wskit@v0.4.0
```

## Minimal Chi Composition

```go
log, err := logkit.New(
    logkit.WithServiceName("api"),
)
if err != nil {
    return err
}
defer log.Close()

slogLog := slog.New(logkit.SlogHandler(log))

r := chi.NewRouter()
r.Use(middleware.RequestID())
r.Use(middleware.Logger(slogLog, nil))
r.Use(middleware.Recoverer(slogLog))
r.Use(middleware.SecurityHeaders())
r.Use(middleware.ContextTimeout(10 * time.Second))
r.Use(metrics.Middleware(nil, chiPathFromRequest, metrics.WithLogger(slogLog)))
r.Use(localization.Middleware(bundle))

r.Get("/health", httputil.HealthHandler(nil))
r.Get("/metrics", promhttp.Handler().ServeHTTP)

r.Group(func(r chi.Router) {
    r.Use(jwtkit.JWTAuth(jwtSvc, jwtkit.WithLogger(slogLog)))
    r.Get("/users/{id}", getUserHandler(pool, cache, l1))
})
```

Full working examples live in [`go-kit/examples`](https://github.com/wahrwelt-kit/go-kit/tree/main/examples).

## Examples

| Example | Router | Kits used |
| --- | --- | --- |
| [chi-rest](https://github.com/wahrwelt-kit/go-kit/tree/main/examples/chi-rest) | chi | logkit, httpkit, metrics, localization, pgkit, jwtkit, cachekit |
| [chi-realtime](https://github.com/wahrwelt-kit/go-kit/tree/main/examples/chi-realtime) | chi | logkit, httpkit, wskit, cachekit |
| [gin-rest](https://github.com/wahrwelt-kit/go-kit/tree/main/examples/gin-rest) | gin | logkit, pgkit, jwtkit, cachekit |
| [gin-realtime](https://github.com/wahrwelt-kit/go-kit/tree/main/examples/gin-realtime) | gin | logkit, wskit, cachekit |

## Design Principles

- **Small modules** - each kit owns one backend concern and keeps its public API narrow.
- **Independent releases** - modules have separate `go.mod`, semantic versions, CI, and tags.
- **Standard boundaries** - logging integration uses `log/slog`; HTTP middleware uses `net/http`.
- **Functional options** - configuration style is consistent across the kits.
- **Consumer-side interfaces** - applications define interfaces at their boundaries instead of importing broad kit interfaces.
- **Production defaults** - fail-fast config validation, context-first APIs, strict parsing, safe redaction, and focused test coverage.
