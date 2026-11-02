# Feature-based Architecture & Language / Framework Selection

> **Status:** reference · **Verified:** 2026-10-31 · **Scope:** web product architecture, not a mandate to use multiple languages. **Recheck:** framework/runtime release, production profiling, team skills, deploy topology, security/compliance requirements.  
> **Important:** **TypeScript, Go, Rust, Python... are languages; Next.js is a React web framework**, not a substitute for a language.

## Question / Recommendation

**Chọn công nghệ theo capability + constraints đã xác thực**, không chia FE/BE/worker thành 3 ngôn ngữ mặc định. Nếu một **Next.js + TypeScript + Postgres modular monolith** đáp ứng mọi acceptance/NFR, giữ nguyên. Go/Rust/Python chỉ được đưa vào khi feature thực sự cần hệ sinh thái, performance, isolation hoặc operational model của nó.

## Decision factors (theo thứ tự)

1. **Correctness & boundary:** business invariants, data consistency, authz, privacy, transactional semantics.
2. **Feature fit & interoperability:** framework capabilities, libraries maintained, SDK availability, integration surface.
3. **Operational fit:** hosting/runtime timeouts, deployments, startup/memory, telemetry, team/oncall.
4. **Performance proven:** p95/p99, throughput, memory/CPU profile **trong workload thực**, không tin benchmark chung.
5. **Maintainability & talent:** skill hiện có, type safety, complexity, tests, upgrade/incident burden.
6. **Reversibility & cost:** service count, infra bills, cross-language contracts, CI/build time, rollout/rollback.

Không tự mặc định Rust luôn nhanh hơn Go/Node trong mọi tác vụ; I/O thường bị ràng buộc bởi DB/network. Không dùng benchmark microservice không đại diện để tách service.

## Strengths, trade-offs và điểm dùng hợp lý

| Công nghệ | Điểm mạnh điển hình | Phải trả giá | Phù hợp khi |
| --- | --- | --- | --- |
| **TypeScript + Node.js** | Chia sẻ types, broad web/SDK ecosystem, full-stack developer velocity | Types bị erase tại runtime; cần runtime validation; CPU-heavy code block event loop nếu thiết kế sai | CRUD/API orchestration, business workflows, integrations, dashboards |
| **Next.js (React + TS)** | File-based routing, Server Components, built-in rendering, integrated web/BFF patterns | Host-dependent runtime limits, caching/rendering mental model, không là durable job engine | SEO, authenticated app UI, dashboards, user journeys, forms |
| **Go** | Goroutines, channels, static binaries, tooling/diagnostics, server/network concurrency đơn giản | GC cần profile, type/domain ergonomics tùy team, thêm service tăng ops | High-concurrency API/worker/network service **khi đã có lý do tách** |
| **Rust** | Ownership/memory safety, kiểm soát allocation, hiệu năng dự đoán hơn cho hot path | Learning/build/borrow-checker complexity, ffi/debug burden | CPU/memory-constrained worker, parser, media processing, native/WASM library được benchmark |
| **Python** | Mature AI/data/scientific libraries, rapid experiments, orchestration | Dynamic runtime/runtime validation, performance phụ thuộc workload, production packaging | ML inference/tooling, data pipelines, specialized scientific tasks |
| **Java/Kotlin** | JVM ecosystem, mature backends, observability, transactions | JVM operational complexity, startup/memory tuning ở một số môi trường | Large enterprise/backend ecosystems và team đã có JVM |
| **C#/.NET** | Mature typed services, enterprise tooling, cross-platform backend | Runtime/ecosystem expertise và infra choices | .NET-centric team, APIs, services, business systems |
| **SQL/PostgreSQL** | Constraints/transactions/indexes rất mạnh, consistency | Complex query/planning/tuning, không thay service API/auth | Data integrity, transactional feature, search/filter/aggregations trước khi thêm engine |

**Anti-pattern:** chọn polyglot architecture để “dùng đúng điểm mạnh” trước khi có workload; điều này tạo mới contracts, deployments, monitoring, consistency problems, retraining và chi phí review.

## Feature-based (vertical slice), framework aware

**Tư duy:** Feature ownership = business capability + entrypoints + domain policy + persistence/tests liên quan. Shared chỉ khi **thực sự** được 2+ feature tái sử dụng; không chuyển business logic vào `utils/` khổng lồ.

### Next.js/TypeScript — gợi ý, không ép scaffold

```text
src/
  app/                     # framework routing/layout only
    (app)/orders/page.tsx  # route entrypoint
  features/
    orders/
      components/          # chỉ nếu nhiều components
      service.ts           # use-cases / authorization boundary
      data.ts              # DB operations / API adapter
      schema.ts            # runtime validation + types
      *.test.ts            # tests gần code
    billing/
      ...
  shared/                  # only truly cross-feature primitives
```
- Với app nhỏ, **colocate ngay trong `app/` hoặc feature folder hiện hữu**; không lập đủ tất cả subfolders trước. Tránh import feature khác thông qua private paths; dùng exported contract khi cần.
- `app/` theo Next.js conventions, route groups `(name)` **không** đi vào URL; route handlers/server actions vẫn phải kiểm authz.
- Không ép cả UI/domain/db nằm chung file; phân lớp **trong feature** nếu boundary giúp kiểm thử/maintain.
- TypeScript compile-time type **không thay runtime schema validation** đối với input, DB/API/LLM output.

### Go — giữ package boundary gắn capability

```text
cmd/server/                  # only when binary entrypoint needed
internal/
  orders/                   # handler + use-case + repository interfaces as justified
  billing/
```
Đừng tạo "clean architecture 8 layers" hoặc interface cho từng concrete type nếu không có consumer/test need; package seams và dependency direction vừa đủ.

### Rust — module/crate theo deploy & reuse needs

```text
src/
  main.rs                   # single binary when one service
  orders.rs                 # feature module
  billing.rs
```
`crates/` workspace chỉ khi có independent binaries/libraries cần reuse. Rust thường hợp **hot-path module** hơn là backend microservice mới khi overhead vượt giá trị.

### Python — package theo pipeline/capability

```text
app/
  inference/
  ingestion/
tests/
```
Phân định model/evals, transformations và I/O boundaries; dùng typed validation tại dữ liệu rời hệ thống. Không ép thay framework gốc nếu có thể isolate worker.

## Decision examples by feature (không có winner cố định)

| Capability | Default | Rẽ nhánh khi |
| --- | --- | --- |
| Content/CMS/SEO landing | Next.js + TS | static site/framework đơn giản hơn nếu runtime không cần |
| Signup, profile, billing, SaaS admin | Next/TS + Postgres | existing backends/JVM/.NET có ownership tốt hơn |
| Payments/webhook | App/TS + transactions/idempotency | dedicated worker khi callbacks/latency require it |
| Multi-tenant high concurrency API | Current stack + load test | Go service nếu benchmark + operational SLO chứng minh |
| CPU-heavy image/video transformation | Background worker hiện có | Rust/native library nếu measured CPU/memory bounds |
| ML scoring / embeddings / data ETL | Python worker/inference service | TS integration only nếu model hosted API đáp ứng |
| Realtime stream/notifications | SSE/polling first | Go/WebSocket service khi host/runtime constraints cần |
| Advanced search/ranking | Postgres FTS + index | dedicated search/vector service khi retrieval eval thắng |
| Security-sensitive parser/sandbox | Mature audited package | Rust memory-safe code nếu parser custom là bắt buộc |

**Cross-language break-even:** chỉ tách khi (benefit measured) > (interop contracts + extra deploy + observability + data consistency + incident + maintenance). Khi tách: explicit API/event schema, authn/authz, versioning, observability và rollback.

## Validation plan for an architecture recommendation

- Xác nhận target repo context + existing conventions; đo baseline **p95/p99, errors, CPU, memory, DB query, infra bill** nếu performance là lý do.
- Chọn một feature đại diện và 2 phương án tối đa; lập spike giới hạn, benchmark trên input giống prod; kiểm correctness trước perf.
- Verify deployment limitations của provider, data boundaries, failure handling, migrations/restore và developer ergonomics.
- Viết **target-owned ADR** nếu quyết định ảnh hưởng lâu dài; nếu giữ nguyên stack, chỉ ghi trong issue/plan.
- Giao task vertical slice nhỏ cho **ChatGPT khi có execution tools phù hợp** hoặc **Codex khi có quyền workspace cần thiết**, lựa chọn theo capability/authority trong [TASKS.md](../TASKS.md); review theo risk, ECC HOW.

## Sources (primary)

- [TypeScript — erased types/runtime semantics](https://www.typescriptlang.org/docs/handbook/typescript-from-scratch)
- [Next.js — App Router](https://nextjs.org/docs/app)
- [Next.js — route groups](https://nextjs.org/docs/app/api-reference/file-conventions/route-groups)
- [Next.js — backend for frontend & runtime limits](https://nextjs.org/docs/app/guides/backend-for-frontend)
- [Go — Effective Go & concurrency](https://go.dev/doc/effective_go)
- [Go — GC guide](https://go.dev/doc/gc-guide)
- [Rust — ownership](https://doc.rust-lang.org/stable/book/ch04-01-what-is-ownership.html)
