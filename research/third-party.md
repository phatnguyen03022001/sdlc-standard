# Third-party — Build-vs-Buy, Vendor Risk & Exit Strategy

> **Status:** reference · **Verified:** 2026-10-31 · **Scope:** managed services, SDKs, APIs, hosted scripts, OSS dependencies for web apps. **Recheck:** before vendor selection, renewal, policy/pricing change, critical vulnerability, major SDK/API change, new data geography.  
> **Non-goal:** bảng xếp hạng nhà cung cấp, giá gói hoặc giới hạn quota cố định (phải đọc terms tại thời điểm mua).

## Question / Default

**Build vs buy cho từng capability, không mua cả platform vì một feature.** Ưu tiên thư viện/runtime đã dùng hoặc dịch vụ managed có contract rõ, quyền dữ liệu chấp nhận được và exit khả thi. Vendor không phải source of truth của business rule; target giữ data/control boundaries.

## Capability decision map

| Capability | Lựa chọn tối giản cần đánh giá | Lý do dùng managed | Red flags / exit |
| --- | --- | --- | --- |
| Hosting/CDN | Host hỗ trợ framework + CI/CD đang có | Không tự vận hành host/SSL | egress lock-in, runtime limits, hard-to-export builds |
| DB & storage | Managed Postgres / S3-compatible storage | Backup, operations, HA | PITR/restore không rõ, portability, export cost |
| Auth | Managed identity hoặc established auth library | MFA, recovery, provider auth | tenant isolation, data export, provider outage/revocation |
| Email/SMS/push | Transactional provider khi feature cần | Deliverability, retries, bounces | DNS reputation, PII retention, regional constraints |
| Payments/billing | Regulated processor theo quốc gia/business | PCI scope reduction, settlement | webhook reliability, disputes, refunds, fees, portability |
| Search | DB built-in trước | specialized relevance/filtering | synchronization lag, expensive indexing, permissions |
| Analytics/telemetry | Minimal events / own logs trước | Dashboards, funnels, ops | browser scripts/PII leakage, sampling costs |
| AI/LLM | API khi có eval và unit economics | Faster model access | PII training/data policy, latency, cost ceilings, fallback quality |
| Files/Media/CDN | Provider supports signed URL, upload/lifecycle | Bandwidth/media transformations | transformation lock-in, public ACL, export |
| Queue/cron | Native job support trước | Durable retries/scheduling | semantics, duplicate delivery, missing DLQ/replay |
| OSS packages | Built-in/maintained library before new dependency | Reuse proven implementation | abandonware, license conflicts, transitive vulnerabilities |

**Ví dụ provider chỉ để tra cứu, không phải endorsement:** Vercel/Cloudflare/Render/AWS cho hosting; Supabase/Neon/RDS cho Postgres; Stripe hoặc processor vùng được phép cho payments; Resend/SES cho email; Sentry/OpenTelemetry cho errors/tracing. **Không ghi bảng giá hoặc claim SLA từ ví dụ này.**

## Evaluation scorecard (hard gates trước, score sau)

### Hard gates — bất kỳ FAIL nào cũng dừng việc chọn

- [ ] Có legal right / jurisdiction / license phù hợp; terms, DPA và retention chấp nhận được.
- [ ] Security & data handling: nơi lưu, subprocessors, encryption, access, audit, incident notification, account deletion/export.
- [ ] API contract và failure behavior đáp ứng user flow; có timeout, backoff/rate limits, retries, idempotency/webhook signature nếu cần.
- [ ] Có thể quan sát service errors và restore/exit theo RPO/RTO của feature, không phụ thuộc dashboard vendor như bằng chứng duy nhất.
- [ ] Không nhúng untrusted third-party script có quyền DOM/cookie không cần thiết; kiểm consent/PII/CSP/SRI khi áp dụng.
- [ ] Không có license/vulnerability blocker không được chấp nhận có thẩm quyền.

### Nếu gates PASS: so sánh theo trọng số của feature

| Factor | Cần thu thập (không bịa) |
| --- | --- |
| Functional fit | use-case demo, API/spec coverage, authz, webhook delivery |
| Reliability | published SLA **và** observed error rate, incident history, support escalation |
| Performance | end-to-end p95/p99 từ môi trường giống prod, not vendor benchmark |
| Total cost | unit pricing, seats, min spend, overage, egress, logs, support, migration/exit |
| Developer experience | SDK maturity, docs, typing, local emulator, sandbox, version policy |
| Security/privacy | threat model delta, secrets, legal region, data residency, audit trail |
| Reversibility | export formats, switching time, contract exit, data portability |
| Operation | provider monitoring, on-call, alert, ownership, kill switch/feature flag |

Lưu câu trả lời và các nguồn **trong một ADR/spec của feature khi quyết định khó đảo ngược**; không lập một Markdown file per provider/SDK. Trọng số tùy feature, không có vendor ranking chung mọi dự án.

## Integration acceptance (đi qua deployment gates)

1. **Boundary:** adapter/client ở feature boundary, typed request/response, runtime validation; đừng thêm wrapper layer nếu API đơn giản.
2. **Secret handling:** server-side only, least privilege/rotation, phân tách test/staging/prod; không log credentials.
3. **Failures:** timeouts, error taxonomy, circuit breaker chỉ khi có nhu cầu; rate limit & retry với jitter/budget; tránh retry tạo double charge.
4. **Webhooks:** verify signature/replay window, durable idempotency key + transactional effect, retry test, reconciliation nếu nghiệp vụ tiền bạc.
5. **Data:** biết trường nào rời hệ thống, có consent/legal basis, retention, deletion và sample redacted logs.
6. **Testing:** sandbox + contract tests + injected 4xx/5xx/timeout; verify degraded mode và callback duplicates.
7. **Deploy:** feature flag/kill switch nếu rủi ro; monitoring/error budget, vendor outage runbook và cách rollback.
8. **Exit:** export test dữ liệu, mapping, operator, downtime và chi phí ước tính; không dời dữ liệu mà không có kiểm tra integrity.

## Open-source / JS supply chain

- Dependency tree + lockfile, automated vulnerability alerts/SCA, security fix policy, license scan khi phát hành.
- Browser third-party JS có quyền với DOM nên cần đánh giá CSP, script sources, SRI/sandbox phù hợp và disclosure; **server-side analytics không tự khiến các script khác an toàn**.
- Cẩn thận package forks, abandoned dependencies, transitive SDK permissions; giảm số dependency mới, không tự rewrite crypto/auth/payment.
- NIST SSDF và OWASP quản lý risk; không áp đặt phải cài mọi scanner/SBOM tool khi không có yêu cầu.

## Đầu ra quyết định (ngắn)

```md
Feature & need: ...
Buy/build candidates: existing | vendor A | vendor B
Hard gates: PASS/FAIL + sources
Key trade-off: security/privacy | correctness | cost | failure mode | exit
Observed test/bench: ...
Decision + owner + date + revisit trigger: ...
```

## Sources (primary)

- [NIST SP 800-218 SSDF](https://csrc.nist.gov/pubs/sp/800/218/final)
- [OWASP Third Party JavaScript Management](https://cheatsheetseries.owasp.org/cheatsheets/Third_Party_Javascript_Management_Cheat_Sheet.html)
- [OWASP API Security Top 10](https://api-security.owasp.org/editions/2023/en/0x11-t10/)
- [GitHub dependency graph](https://docs.github.com/en/code-security/concepts/supply-chain-security/dependency-graph-data)
- [SLSA supply chain provenance](https://slsa.dev/spec/v1.1/)
