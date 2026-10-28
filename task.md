# task.md — Posture AI Customer Support Copilot

Task tracker derived from `CLAUDE.md`. Check items off as they land; keep each task small enough for a single PR.

**Legend:** `[x]` done · `[ ]` todo · **P0** next up · **P1** soon · **P2** later

---

## Phase 1 — MVP (Complete)

### Auth & RBAC
- [x] `POST /auth/login`, `/auth/signup`, `/auth/refresh`, `GET /auth/me`
- [x] JWT access + refresh tokens, bcrypt password hashing
- [x] `require_role()` dependency (admin / agent / customer)
- [x] Login + signup pages with role selector
- [x] `useAuth` / `useUser(requiredRole?)` hooks with role-based redirects
- [x] 13 tests

### AI Chatbot (RAG)
- [x] `POST /chat/completions`, `GET /chat/conversations/{id}/history`
- [x] `WS /ws/chat/{conversation_id}` with JWT query-param auth
- [x] `rag_service.retrieve_context` (embed + pgvector) and `generate_answer`
- [x] Chat page: WebSocket + REST fallback, citations, "Talk to Agent" after 3 AI attempts
- [x] 4 tests

### Knowledge Base
- [x] Document CRUD (soft delete + chunk removal), collections, URL ingest
- [x] Ingestion worker: `parse_pdf` → `chunk_text` → embeddings → `knowledge_chunks`
- [x] Admin KB page: upload modal, URL ingest, document table, delete
- [x] 9 tests

### Smart Ticketing
- [x] Ticket CRUD + messages, auto-set `closed_at`
- [x] Auto-assign to least-loaded available agent
- [x] Ticket list (status filter), detail page, create modal, status actions
- [x] 9 tests

### Agent Copilot
- [x] `POST /copilot/suggest-reply`, `/copilot/summarize`, `/copilot/retrieve-kb`
- [x] Copilot buttons on ticket detail and agent pages
- [x] 6 tests

### Admin Panel & Analytics
- [x] Agents list/update, AI config read/update (in-memory)
- [x] `GET /analytics/dashboard` metrics
- [x] Admin dashboard, agents, analytics, settings pages
- [x] 11 admin tests + 4 analytics tests

### Platform
- [x] Health check endpoint (2 tests)
- [x] RequestContextMiddleware (X-Request-ID, timing logs)
- [x] Structured `AppException` error responses
- [x] Redis client with graceful degradation
- [x] Initial Supabase migration (`backend/supabase/migrations/001_initial_schema.sql`)
- [x] Dockerfile (backend) + standalone Next.js build

---

## Phase 2 — Post-MVP Hardening

### Storage & Ingestion
- [ ] **P0** Store uploaded documents in S3 (`AWS_S3_BUCKET`), save `storage_key` on `knowledge_documents`
- [ ] **P0** Move ingestion off the request path: enqueue job in Redis/SQS, run `ingestion_worker` as a consumer
- [ ] **P1** Retry + dead-letter handling for failed ingestions; surface `failed` reason in KB UI
- [ ] **P1** Re-ingest / re-embed a document on demand
- [ ] **P2** Support DOCX, Markdown, and HTML sources in addition to PDF

### Real-Time Chat
- [ ] **P0** Stream AI responses token-by-token over WebSocket using `chat_completion_stream`
- [ ] **P1** Typing indicators and delivery state in `ChatWindow`
- [ ] **P1** Live handoff: push escalated conversations to agent inbox in real time
- [ ] **P2** Redis pub/sub for WebSocket fan-out across multiple backend instances

### AI Config & API Keys
- [ ] **P0** Persist AI config to the database (currently in-memory, lost on restart)
- [ ] **P1** API key endpoints: create (show once, store hash), list, revoke — tenant-scoped
- [ ] **P1** Wire the settings page API-keys placeholder to the new endpoints
- [ ] **P1** API-key auth path alongside JWT for developer access

### Analytics
- [ ] **P1** Time-series endpoint (tickets created/resolved per day, response times)
- [ ] **P1** Replace chart placeholders on `/admin/analytics` (honor time range filter)
- [ ] **P2** Per-agent performance metrics (handled, avg resolution, CSAT)

### Security & Compliance
- [ ] **P0** Verify RLS policies on every tenant-scoped table; add tests for cross-tenant access
- [ ] **P1** Write to `audit_logs` for auth, ticket status changes, KB deletes, config changes
- [ ] **P1** Rate limiting on auth and chat endpoints (Redis-backed, degrade gracefully)
- [ ] **P1** Restrict signup role selection (admin/agent via invite only)
- [ ] **P2** Refresh-token rotation and revocation list

### Testing & Quality
- [ ] **P1** Frontend tests for hooks (`useUser`, `useWebSocket`) and key pages
- [ ] **P1** Backend tests for WebSocket chat and ingestion worker edge cases
- [ ] **P2** End-to-end smoke test (login → chat → escalate → ticket → copilot reply)
- [ ] **P2** CI pipeline: lint, type-check, pytest, `npm run build` on every PR

### Deployment
- [ ] **P1** Frontend Dockerfile; verify `docker-compose.yml` runs full stack locally
- [ ] **P1** AWS: ECS Fargate services, ALB (WebSocket-sticky), S3 bucket, secrets in SSM/Secrets Manager
- [ ] **P2** Alerting on health check failures and ingestion error rate

---

## Phase 3 — Expansion (Backlog)

- [ ] Channels: WhatsApp, SMS, Slack, Microsoft Teams
- [ ] Voice / video support
- [ ] CRM integrations
- [ ] Sentiment analysis on conversations and tickets
- [ ] Automation workflows (rules, triggers, SLA escalations)
- [ ] Multi-language support
- [ ] Dark mode (extend Euron tokens)
- [ ] Native mobile app

---

## Working Rules (from CLAUDE.md)

- Routes thin → services hold logic → repositories own DB access, always `tenant_id`-scoped
- Schema changes only via Supabase migrations, with RLS on tenant tables
- Mock Supabase / EURI / Redis in tests; cover RBAC rejections for every admin-only endpoint
- Follow the Euron design system strictly; no external UI libraries, no emojis in UI
- Log every prompt to `prompts/prompt-history.md`
