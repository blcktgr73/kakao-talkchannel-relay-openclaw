# Kakao TalkChannel Relay Server

Read AGENTS.md for coding style, commit conventions, and testing guidelines.

## Project

멀티테넌트 메시지 릴레이 서버 — 공유 카카오톡 채널을 여러 OpenClaw 인스턴스에 연결.
웹훅 라우팅, 메시지 큐잉, 페어링, 세션 관리를 처리합니다.

## Quick Commands

```bash
# Infrastructure
make docker-up        # Start PostgreSQL + Redis
make docker-down      # Stop containers
make setup            # docker-up + db-migrate

# Go Server
make dev              # Run server (go run ./cmd/server)
make build            # Compile binary
go test ./...         # Run Go tests

# Frontend (Bun)
bun run build:admin   # Build admin UI
bun run build:portal  # Build portal UI
bun run build:all     # Build admin + portal + server
bun test              # Run TS tests

# Database (Drizzle)
make db-migrate       # Run migrations
make db-generate      # Generate migration
make db-studio        # Drizzle Studio UI
make db-reset         # Reset database

# Code Quality
make check            # Biome check
make lint             # Biome lint
make format           # Biome format
```

## Tech Stack

**Backend (Go 1.25):**
- chi v5 (HTTP router)
- sqlx + lib/pq (PostgreSQL)
- go-redis v9 (Redis)
- zerolog (structured logging)
- testify (testing)

**Frontend (Bun + TypeScript):**
- React 19, React Router 7
- Tailwind CSS 4, Lucide icons
- Drizzle ORM, Zod validation
- Biome (lint & format)

**Infrastructure:**
- PostgreSQL 16, Redis 7
- Docker Compose (dev), Cloud Run (prod)
- Multi-stage Dockerfile (Alpine)

## Architecture

```
cmd/server/main.go          # Entrypoint (chi router setup)
internal/
├── handler/                # HTTP handlers
│   ├── kakao.go            # Kakao webhook (/kakao-talkchannel/webhook)
│   ├── openclaw.go         # OpenClaw API routes
│   ├── admin.go            # Admin UI API
│   ├── portal.go           # Portal UI API
│   ├── session.go          # Session management (/v1/sessions)
│   └── events.go           # SSE endpoint (/v1/events)
├── service/                # Business logic
│   ├── conversation.go     # Conversation management
│   ├── pairing.go          # Pairing code generation & verification
│   ├── message.go          # Message handling
│   └── session.go          # Session lifecycle
├── repository/             # Data access layer
├── middleware/              # Auth, rate limit, CSRF, security headers
├── model/                  # Data models & enums
├── config/                 # Environment config & constants
├── database/               # DB connection
├── redis/                  # Redis client wrapper
├── sse/                    # Server-Sent Events broker (Redis Pub/Sub)
├── audit/                  # Audit logging
├── util/                   # Crypto, validation, encryption
└── errors/                 # Custom error types

admin/src/                  # Admin UI (React SPA)
portal/src/                 # Portal UI (React SPA)
drizzle/migrations/         # SQL migration files (0000-0008)
public/                     # Built frontend assets
```

### Key Patterns

- **Service Layer**: handler → service → repository
- **Long Polling**: OpenClaw 메시지 수신
- **Idempotency**: source event ID로 중복 방지
- **Multi-Tenant**: Account 기반 격리
- **TTL Cleanup**: 만료 메시지/페어링 코드 자동 삭제
- **Rate Limiting**: Redis 기반, per-account + IP

## Environment

`.env.example` 참조. 필수 변수:
- `DATABASE_URL` — PostgreSQL 연결 문자열
- `REDIS_URL` — Redis 연결 문자열
- `ADMIN_PASSWORD_HASH` — bcrypt 해시 (v0.3.0+ breaking change)
- `ADMIN_SESSION_SECRET` / `PORTAL_SESSION_SECRET` — 세션 시크릿
- Optional: `KAKAO_SIGNATURE_SECRET`, `ENCRYPTION_KEY`, `PORTAL_BASE_URL`

## Deployment

- **Local**: `make setup && make dev`
- **Production**: `deploy.sh` (Cloud Run, gcloud builds submit)
- **Docker**: Multi-stage Alpine build, non-root user, HEALTHCHECK

## Key Documents

- [AGENTS.md](AGENTS.md) — Coding style, commit conventions, testing
- [docs/architecture.md](docs/architecture.md) — System design & data flow
- [docs/api-spec.md](docs/api-spec.md) — API specification
- [docs/pairing-flow.md](docs/pairing-flow.md) — Pairing flow documentation
- [docs/setup-guide.md](docs/setup-guide.md) — Setup instructions
- [docs/integration-guide.md](docs/integration-guide.md) — OpenClaw integration guide
