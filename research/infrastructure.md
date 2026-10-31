# Infrastructure — Decision Research for Feature-based Web Apps

> **Status:** reference · **Verified:** 2026-10-31 · **Scope:** web app production, từ single product tới multi-service khi có bằng chứng. **Recheck:** workload/SLO/traffic thay đổi, region/data policy, cloud pricing, incident, provider lifecycle hoặc runtime limits.  
> **Non-goal:** không chọn provider hoặc áp IaC/config vào một target repo chưa khảo sát.

## Question

Hạ tầng **ít thành phần nhất** nào đủ đáp ứng availability, latency, data integrity, security, cost và khả năng vận hành của feature? Cái gì là requirement, cái gì chỉ là option?

## Baseline có điều kiện

**Mặc định cho sản phẩm nhỏ–trung bình có nhiều feature:**
```text
Browser/mobile → CDN/TLS → Web application (Next.js/TS hoặc backend đã có)
                               ├─ PostgreSQL (durable source of truth)
                               ├─ Object storage (media/files khi cần)
                               └─ Managed background jobs (nếu request cần async)
Observability: request IDs + error tracking + metrics tối thiểu
CI/CD: build → tests → staging/UAT khi cần → release → smoke → rollback
```
- **Modular monolith theo feature** là điểm khởi đầu. Internal module boundary **không đồng nghĩa** network/microservice boundary.
- Prefer managed components **khi chi phí vận hành/hỗ trợ phù hợp**, không phải vì một provider được quảng cáo “serverless”.
- Thiết kế authz, data ownership, migration, backup và recovery **ngay từ đầu**; không mua multi-region, Kubernetes, Kafka, Redis, service mesh khi chưa có lý do.
- Một Next.js backend-for-frontend **không tự động là hệ thống worker**: giới hạn runtime/deployment (filesystem, functions, WebSockets, timeouts) cần xác minh theo host. Server Actions thích hợp mutations, không dùng làm long-running worker.

## Quyết định theo năng lực / failure mode

| Capability | Mặc định nhỏ nhất | Kích hoạt thêm khi | Xác minh trước khi chọn |
| --- | --- | --- | --- |
| Hosting/compute | Một deploy target app; đủ TLS + env secrets + rollback | p95/p99/SLO, runtime cap hoặc region policy không đáp ứng | concurrency, cold start, request timeout, logs, egress, build/deploy limits |
| Database | PostgreSQL managed + migrations + constraints | cần read scale, HA hoặc partition **đo được** | RPO/RTO, pooling, connection cap, backup restore, PITR, failover, lock/migration |
| Sessions/auth | managed auth hoặc thư viện đã có; server-side authorization | regulatory/security yêu cầu phương án riêng | token expiry, revocation, MFA, audit, cookie security, tenant isolation |
| Object storage | S3-compatible/managed buckets, signed access | file lớn, CDN, legal retention | ACL, size limits, malware scanning, encryption, lifecycle, signed URLs |
| Caching | HTTP/CDN caching và query/index trước | repeated high-cost reads hoặc traffic hotspot | invalidation, stale reads, cache keys bao gồm tenant, PII exposure |
| Async jobs | provider job/scheduler hoặc DB-backed jobs khi đủ | tác vụ vượt timeout, retries, external rate limits | delivery semantics, dedup, idempotency, DLQ/replay, timeouts, worker budget |
| Messaging/event bus | transaction + direct code path | nhiều consumers/bounded contexts cần durable delivery | transactional outbox, ordering, replay, poison messages, schema versioning |
| Realtime | polling/SSE phù hợp | full duplex hoặc many concurrent sessions thật sự cần | long-lived connection support, auth per channel, reconnection, pricing |
| Search | SQL indexes/FTS trước | semantic relevance, large inverted index hoặc fuzzy requirements đo được | freshness lag, ranking eval, index cost, ACL filtering |
| Queue/cache service | Không thêm mặc định | throughput/coordination vượt khả năng thành phần hiện hữu | TTL, eviction, HA, loss/replay, security network exposure |
| Observability | health, errors, latency, basic logs, request IDs | nhiều service hoặc root-cause không theo dõi được | log retention, PII redaction, sampling, cardinality, alerts |
| CDN/WAF/rate limits | TLS/CDN + API throttling theo risk | abuse/public endpoints/high traffic | bypass paths, per-user/IP correctness, false positives, request body limits |
| Multi-region/HA | Một region có backup và runbook rõ | RTO/RPO/business impact bắt buộc | data replication consistency, failover test, DNS, cost, data residency |

### Những ràng buộc phải kiểm thử chứ không đoán

1. **Request life cycle:** max duration, streaming, WebSockets, background execution của **host thực tế**; không suy từ framework.
2. **Durable data:** migration backward compatibility, constraints, data retention và restore test có bằng chứng. Migration rollback không luôn hoàn nguyên được dữ liệu: phân biệt code rollback và data recovery.
3. **Isolation:** tenant authorization ở tất cả điểm đọc/ghi; object storage không lộ public; cache không trộn tenant.
4. **Queue semantics:** mặc định giả định **at-least-once** nếu không có hợp đồng khác; consumer idempotent, bounded retries và dead-letter/replay khi rủi ro yêu cầu.
5. **Observability:** từ user action tới DB/third party có correlation ID; không log tokens/PII.
6. **Costs:** tính compute + DB + storage + egress + bandwidth + logs + CI + backup + vendor support. Không chỉ free-tier giá quảng cáo.

## Evolution path và ngưỡng

**Stage 0:** one app + Postgres; SLO và tracing/log đủ xem nguyên nhân lỗi.  
**Stage 1:** tối ưu queries, indexes, CDN/cache và background task theo profile thực; đo trước/sau.  
**Stage 2:** tách worker hoặc search service **theo queue/runtime constraint**, chưa tách domain ownership.  
**Stage 3:** chỉ tách service theo bounded context khi đã có independent deploy/scale, data ownership, team hoặc isolation requirement và chấp nhận chi phí network + observability + consistency.  
**Stage 4:** HA/multi-region khi business yêu cầu RTO/RPO có đo, không dựa vào danh xưng enterprise.

**Decision gate:** Current baseline fails which exact SLO/invariant? Evidence (trace/load/incident)? Two smaller fixes tried? New failure modes, security impact, monthly cost and rollback plan?

## Reference feature scenarios (illustrative, not requirements)

| Feature | Baseline | Chỉ tách thêm nếu |
| --- | --- | --- |
| Login/profile/admin | App server + managed auth + DB | compliance/tenant isolation thực tế |
| Image upload | Signed upload + object storage + metadata | CPU/image scan/processing vượt request lifecycle → worker |
| Search/catalog | DB filters/FTS, index | relevance/latency or corpus size measured → dedicated search |
| AI job | API orchestration + budget/timeout | duration/retry/load → queue worker; evals & PII filtering mandatory by risk |
| Payments/webhooks | DB transaction + dedup/idempotency record | webhook retries require durable queue/replay |
| Dashboard/analytics | DB/report queries + cache | reporting load impacting OLTP → separate read store/warehouse |
| Real-time collaboration | Poll/SSE initially | bidirectional presence/high fanout → dedicated realtime technology |

## Sources (primary)

- [AWS Well-Architected Framework — six pillars](https://docs.aws.amazon.com/wellarchitected/latest/framework/) — quality trade-off, not mandatory AWS adoption.
- [Next.js Backend for Frontend guide](https://nextjs.org/docs/app/guides/backend-for-frontend) — Route Handlers and deployment constraints.
- [OWASP API Security Top 10](https://api-security.owasp.org/editions/2023/en/0x11-t10/) — authorization, resource consumption and unsafe integrations.
- [OpenTelemetry concepts](https://opentelemetry.io/docs/concepts/) — traces/metrics/logs as optional diagnostic components.
- [PostgreSQL documentation](https://www.postgresql.org/docs/current/index.html) — transactions, indexes, lock/migration details.
- [NIST SSDF](https://csrc.nist.gov/pubs/sp/800/218/final) — secure changes throughout SDLC.
