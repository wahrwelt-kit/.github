# wahrwelt-kit

Focused Go modules for backend services.

Each kit is versioned and released independently. The [`go-kit`](https://github.com/wahrwelt-kit/go-kit) repository contains runnable examples showing how the modules fit together.

## Kits

| Kit | Latest | Description |
| --- | --- | --- |
| [go-logkit](https://github.com/wahrwelt-kit/go-logkit) | `v0.7.0` | Structured logging on zerolog: nested field redaction, async writer, hooks, context fields, `log/slog` bridge |
| [go-httpkit](https://github.com/wahrwelt-kit/go-httpkit) | `v0.8.0` | HTTP helpers: strict JSON decoding, errors, rendering, validation, pagination, SSE, health, middleware |
| [go-httpkit/metrics](https://github.com/wahrwelt-kit/go-httpkit/tree/main/metrics) | `v0.8.0` | Separate Prometheus middleware module with bounded HTTP method labels |
| [go-httpkit/localization](https://github.com/wahrwelt-kit/go-httpkit/tree/main/localization) | `v0.8.0` | Separate go-i18n request localization module |
| [go-pgkit](https://github.com/wahrwelt-kit/go-pgkit) | `v1.5.0` | PostgreSQL pool setup, pgx helpers, transactions, goose and golang-migrate runners |
| [go-jwtkit](https://github.com/wahrwelt-kit/go-jwtkit) | `v0.7.0` | JWT access/refresh pairs, symmetric and asymmetric keys, strict claims, JWK/JWKS helpers, revocation |
| [go-cachekit](https://github.com/wahrwelt-kit/go-cachekit) | `v0.8.0` | Redis JSON cache with singleflight, SIEVE in-memory cache, TTL single-value cache, key-value and Pub/Sub helpers |
| [go-wskit](https://github.com/wahrwelt-kit/go-wskit) | `v0.5.1` | WebSocket and SSE hub on coder/websocket with optional Redis Pub/Sub fan-out |

HTTP helpers, cache values, structured log fields, and WebSocket event envelopes use `encoding/json/v2`. JWT middleware uses v2 for error responses; token signing and parsing remain with `golang-jwt/jwt/v5`. See each kit's README for JSON compatibility details, including `omitempty` and `omitzero`.

## Install

Requires Go 1.27.2 or newer. Install the modules your application uses:

```bash
go get github.com/wahrwelt-kit/go-logkit@v0.7.0
go get github.com/wahrwelt-kit/go-httpkit@v0.8.0
go get github.com/wahrwelt-kit/go-httpkit/metrics@v0.8.0
go get github.com/wahrwelt-kit/go-httpkit/localization@v0.8.0
go get github.com/wahrwelt-kit/go-pgkit@v1.5.0
go get github.com/wahrwelt-kit/go-jwtkit@v0.7.0
go get github.com/wahrwelt-kit/go-cachekit@v0.8.0
go get github.com/wahrwelt-kit/go-wskit@v0.5.1
```

## Examples

| Example | Router | Kits used |
| --- | --- | --- |
| [chi-rest](https://github.com/wahrwelt-kit/go-kit/tree/main/examples/chi-rest) | chi | logkit, httpkit, metrics, localization, pgkit, jwtkit, cachekit |
| [chi-realtime](https://github.com/wahrwelt-kit/go-kit/tree/main/examples/chi-realtime) | chi | logkit, httpkit, wskit, cachekit |
| [gin-rest](https://github.com/wahrwelt-kit/go-kit/tree/main/examples/gin-rest) | gin | logkit, pgkit, jwtkit, cachekit |
| [gin-realtime](https://github.com/wahrwelt-kit/go-kit/tree/main/examples/gin-realtime) | gin | logkit, wskit, cachekit |

Follow the [go-kit setup instructions](https://github.com/wahrwelt-kit/go-kit#examples) for environment variables and the demo `users` table. From a go-kit checkout, run an example with `go run ./examples/chi-rest` or another directory from the table.

- REST examples require PostgreSQL, Redis, and distinct `JWT_ACCESS_SECRET` and `JWT_REFRESH_SECRET` values of at least 32 bytes each. Both listen on `:8080`.
- Their `/auth/login` endpoint issues demo tokens for a supplied user ID and role without verifying credentials. It is not production authentication.
- Realtime examples require Redis and listen on `:8081`; run one example per port at a time. Both expose `/ws` and `/broadcast`; chi also exposes `/events` for SSE.

## Design Principles

- **Independent modules** - the six kits do not import one another and have separate versions and release cycles.
- **Standard boundaries** - logging integration uses `log/slog`; HTTP middleware uses `net/http`.
- **Functional options** - configure behavior through explicit options.
- **Consumer-side interfaces** - applications define interfaces at their own boundaries.
