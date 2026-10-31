# SDLC — 11 giai đoạn cho Web App production-grade

> **Version:** 1.1 · **Cập nhật:** 2026-10-31 · **Loại:** Reusable checklist / standard template  
> **Phạm vi:** Web app phức tạp, triển khai thực tế, có thể mở rộng cho AI-native apps.  
> **Tính chất:** Khung thực hành nội bộ tham chiếu các chuẩn quốc tế, **không phải** bản sao tiêu chuẩn ISO hay chứng nhận tuân thủ.

## A. Quy tắc sử dụng

1. **11 giai đoạn là các nhóm công việc**, không phải quy trình waterfall bắt buộc. Có thể thực hiện song song và lặp lại theo feature/release.
2. Checklist là **baseline có thể điều chỉnh theo rủi ro**. Bắt buộc duy trì các kiểm soát cốt lõi về correctness, security, privacy, testing, vận hành. Không triển khai queue, microservices, distributed locks, canary, AI infrastructure… nếu chưa có yêu cầu chứng minh được.
3. Mỗi giai đoạn có **Deliverables** (đầu ra) và **Gate** (điều kiện qua giai đoạn). Gate chỉ PASS khi có **bằng chứng kiểm chứng**; có thể ghi N/A + lý do với mục không áp dụng.
4. **UAT và phê duyệt phát hành diễn ra trước production**. Sau triển khai cần smoke test, theo dõi và sẵn sàng rollback.
5. Một feature không nhất thiết tạo 11 bộ tài liệu riêng; dùng tài liệu tối thiểu, liên kết các quyết định, task, test và release với nhau. Ưu tiên **KISS, root cause, smallest correct solution**.

**Luồng tham chiếu:** Discovery → Requirements → Architecture → Planning → Design → Implementation → Testing & QA → Acceptance & Readiness → Deployment & Release → Operations & Evolution → Retirement.

---

## B. Checklist 11 giai đoạn

### 1. Discovery — Khám phá vấn đề

- [ ] Xác định problem statement và giá trị cần tạo.
- [ ] Xác định stakeholders, owner và nhóm người dùng mục tiêu.
- [ ] Nghiên cứu pain points, user journeys, các giải pháp đang có.
- [ ] Đánh giá tính khả thi kỹ thuật, vận hành, pháp lý và tài chính.
- [ ] Xác định mục tiêu định lượng: KPIs / success metrics.
- [ ] Chốt phạm vi sơ bộ, ràng buộc, giả định và các vấn đề chưa rõ.
- [ ] Ghi rủi ro chính và quyết định **go / no-go**.

**Deliverables:** Problem brief, mục tiêu, scope sơ bộ, feasibility/risk notes.  
**Gate 1 — PASS khi:** Bài toán, người dùng, giá trị và tiêu chí thành công rõ ràng; có người chịu trách nhiệm quyết định.

### 2. Requirements — Đặc tả yêu cầu

- [ ] Functional requirements: hành vi và nghiệp vụ cần hỗ trợ.
- [ ] Non-functional requirements: security, performance, availability, accessibility, maintainability, cost.
- [ ] User stories / use cases, user flows và business rules.
- [ ] Phân quyền, vai trò, boundaries, dữ liệu nhạy cảm và yêu cầu privacy/compliance.
- [ ] Các ràng buộc dữ liệu: nguồn, quyền sở hữu, retention, export/delete.
- [ ] Edge cases, failure modes và hành vi khi dịch vụ phụ trợ lỗi.
- [ ] Acceptance criteria **có thể kiểm thử** cho mỗi yêu cầu quan trọng.
- [ ] Sắp xếp ưu tiên và traceability: requirement ↔ feature/task ↔ test.
- [ ] Xử lý xung đột yêu cầu và xác nhận với stakeholder.

**Deliverables:** PRD/spec tối thiểu, user flows, acceptance criteria, danh sách NFR.  
**Gate 2 — PASS khi:** Scope đủ rõ để thiết kế; yêu cầu quan trọng có tiêu chí nghiệm thu đo/kiểm tra được.

### 3. Architecture — Kiến trúc kỹ thuật

- [ ] System context, boundaries, data flows và các điểm tích hợp.
- [ ] Lựa chọn stack phù hợp năng lực vận hành, team và chi phí.
- [ ] Chọn kiến trúc **đơn giản nhất đáp ứng yêu cầu**; xác định module ownership.
- [ ] Thiết kế API contracts, versioning, validation và error semantics.
- [ ] Data model, indexes, transaction boundaries, integrity và lifecycle.
- [ ] AuthN/AuthZ, threat modeling, least privilege, secrets management.
- [ ] Targets cho latency, throughput, capacity và resilience; caching khi cần.
- [ ] Backup/restore, RPO/RTO (nếu có yêu cầu), failure recovery.
- [ ] Xác định **backward compatibility** cho API/schema và chiến lược migration khi có nhiều version chạy đồng thời.
- [ ] **Khi cần:** background jobs, queue/event-driven, retry, timeouts, deduplication/idempotency, rate limiting; distributed locking **chỉ nếu** không thể giải quyết bằng transaction/constraint/coordination đơn giản hơn.
- [ ] Lưu ADR cho các trade-off có tác động dài hạn, loại trừ overengineering.

**Deliverables:** Architecture overview, API/data contracts, threat/risk notes, ADR chọn lọc.  
**Gate 3 — PASS khi:** Thiết kế đáp ứng NFR quan trọng, rủi ro lớn có biện pháp, phương án triển khai/bảo trì khả thi.

### 4. Planning — Kế hoạch thực thi

- [ ] Roadmap và milestone theo giá trị bàn giao.
- [ ] Chia epic → feature → task có phạm vi nhỏ, độc lập tối đa.
- [ ] Dependency mapping, critical path và giới hạn WIP.
- [ ] Ước lượng effort, ngân sách và năng lực thực tế.
- [ ] Risk register, owner, phương án giảm thiểu.
- [ ] Definition of Ready / Done cho task/feature/release.
- [ ] Chiến lược kiểm thử, release cadence, rollback ownership.
- [ ] Đặt mốc review và cơ chế kiểm soát thay đổi scope.

**Deliverables:** Prioritized backlog, milestones, dependencies, release plan.  
**Gate 4 — PASS khi:** Task có owner, điều kiện hoàn thành rõ, không có blocker chưa được quản lý.

### 5. Design — Thiết kế sản phẩm

- [ ] Information architecture, navigation và nội dung chính.
- [ ] Wireframes/prototypes cho luồng quan trọng.
- [ ] Design system: typography, spacing, tokens, components.
- [ ] Responsive/mobile và interaction design.
- [ ] Accessibility theo mục tiêu phù hợp (ví dụ WCAG 2.2 AA khi áp dụng).
- [ ] Loading, empty, error, permission-denied, offline/retry states.
- [ ] Form validation, feedback, destructive actions, confirmation.
- [ ] Usability checks với người dùng hoặc stakeholders phù hợp.
- [ ] Handoff rõ trạng thái, logic, assets, acceptance behavior.

**Deliverables:** UX flows, design specs/components, prototypes khi cần.  
**Gate 5 — PASS khi:** Luồng chính và các trạng thái lỗi đủ rõ để implement và test; không tồn tại UX blocker nghiêm trọng.

### 6. Implementation — Phát triển

- [ ] Repo, local development setup, conventions và môi trường chạy được.
- [ ] FE/BE theo module boundaries, API contracts và naming nhất quán.
- [ ] Code dùng chung chỉ được extract khi có cùng semantics/contract và owner rõ; bảo toàn consumers khi sửa shared (xem [REPOSITORY.md](REPOSITORY.md)).
- [ ] DB migrations có kiểm thử, giữ data integrity; **expand-and-contract/backward-compatible** khi cần zero-downtime hoặc rolling/canary.
- [ ] AuthN/AuthZ, server-side validation, rate limiting **khi có bề mặt abuse**.
- [ ] Error handling, retries có giới hạn, timeouts và idempotency cho thao tác dễ bị lặp.
- [ ] **Khi cần:** async jobs/queues, dead-letter handling, retry policy, deduplication và observability.
- [ ] Secrets không hard-code; cấu hình tách môi trường; kiểm soát quyền truy cập.
- [ ] Lint, format, typecheck và unit/integration tests phù hợp.
- [ ] Code review hoặc kiểm tra độc lập cho thay đổi có rủi ro.
- [ ] Supply chain: dependency lockfile, SCA, SAST, license compliance; SBOM/provenance khi rủi ro/yêu cầu đòi hỏi.
- [ ] Docs tập trung vào hành vi, cách chạy, migration và quyết định khó.

**Deliverables:** Code, migrations, tests, build artifacts, release notes/diff.  
**Gate 6 — PASS khi:** Build/checks xanh, behavior chính đúng, bảo mật cơ bản và data safety có bằng chứng.

### 7. Testing & QA — Kiểm thử và đảm bảo chất lượng

- [ ] Unit tests cho logic quyết định, boundary/edge cases.
- [ ] Integration tests: DB, API, external services, contract tests.
- [ ] E2E tests cho critical user journeys, không lạm dụng E2E.
- [ ] Regression tests cho bug đã sửa và tính năng bị ảnh hưởng.
- [ ] Truy vết changed surface → direct/transitive consumers → test phù hợp; **không** mặc định mọi thay đổi đều chạy E2E toàn hệ thống và **không** bỏ qua consumer không xác định.
- [ ] Security checks: authz bypass, injection, OWASP risks, dependency vulnerabilities.
- [ ] Performance/load testing theo NFR và quy mô thực tế.
- [ ] Accessibility, responsive, cross-browser/device checks.
- [ ] Test data management: synthetic seeding, masking/anonymization khi dùng dữ liệu thật; **không đưa PII/secrets production vào test mặc định**.
- [ ] Failure/recovery tests: timeouts, retry, duplicate delivery, backup restore (khi áp dụng).
- [ ] Defect triage, severity, regression evidence, quality report.

**Chọn test theo blast radius (WHAT / minimum evidence; ECC sở hữu HOW):**

| Changed surface | Phạm vi kiểm chứng tối thiểu |
| --- | --- |
| Private helper/logic cô lập | Targeted unit/regression tests + static checks phù hợp |
| Shared function/module | Unit/contract tests của shared **và** regression của direct/transitive consumers bị ảnh hưởng |
| API/schema/event/shared contract | Contract + integration tests qua các consumer boundaries; kiểm backward compatibility khi có phiên bản đồng thời |
| AuthZ, thanh toán, dữ liệu/migration, global config/runtime | Security/data integrity/failure-path tests + critical integration/E2E hoặc build checks theo rủi ro |

**Ba lớp kiểm chứng:** (1) vòng sửa nhanh: targeted tests + static checks; (2) gate DONE: affected contract/integration/regression và evidence; (3) release/CI: broader regression suite, critical user journeys và checks theo release risk. **Không chạy full E2E sau từng edit**, nhưng phải chạy suite rộng hơn khi thay đổi có blast radius lớn, dependency graph không đáng tin (dynamic import/DI/config/runtime side effect), thiếu tests cho consumer, hoặc release gate đòi hỏi. Test selection chỉ tối ưu phản hồi, **không chứng minh không có regression** nếu mapping thiếu; không ghi PASS khi thiếu evidence bắt buộc. Xem [TASKS.md](TASKS.md) cho kết quả cần báo cáo.

**Deliverables:** Test results, defect list, coverage theo rủi ro, QA summary.  
**Gate 7 — PASS khi:** Critical paths đã kiểm thử; không còn blocker/critical defect chưa chấp nhận rủi ro hợp lệ.

### 8. Acceptance & Readiness — Nghiệm thu trước phát hành

- [ ] Deploy candidate lên **staging/pre-production** đủ tương đồng để xác minh.
- [ ] UAT theo acceptance criteria với người có thẩm quyền.
- [ ] Xác nhận NFR, business flows và dữ liệu/migration readiness.
- [ ] Security/privacy review ở mức rủi ro phù hợp.
- [ ] Runbook, dashboard, on-call/owner, incident contacts sẵn sàng.
- [ ] Backup, rollback plan và quy trình phục hồi được xác minh.
- [ ] Feature flags, cutover/compatibility plan **nếu áp dụng**.
- [ ] Stakeholder sign-off hoặc quyết định phát hành được ghi nhận.

**Deliverables:** UAT evidence, release readiness checklist, sign-off/decision, rollback plan.  
**Gate 8 — PASS khi:** Đủ điều kiện phát hành; **không deploy production trước UAT/sign-off bắt buộc**.

### 9. Deployment & Release — Triển khai và xác minh production

- [ ] CI/CD build, test, artifact integrity và deployment permissions.
- [ ] Production infra, environment config, secrets, network/access controls.
- [ ] DB migration có thứ tự an toàn, kiểm soát compatibility; expand-and-contract khi cần.
- [ ] Backup/snapshot theo risk plan trước thay đổi có khả năng phá dữ liệu.
- [ ] Release strategy phù hợp: direct, rolling, canary hoặc blue-green.
- [ ] Feature flags/gradual rollout khi rủi ro phát hành đòi hỏi.
- [ ] Production smoke tests và kiểm tra user journeys quan trọng ngay sau deploy.
- [ ] Theo dõi logs, metrics, error rates, latency và business indicators.
- [ ] Rollback/recovery nếu vượt ngưỡng lỗi; ghi lại release decision.

**Deliverables:** Deployment record, release version, smoke-test evidence, monitoring snapshot.  
**Gate 9 — PASS khi:** Production ổn định theo release criteria, rollback path khả dụng, ownership bàn giao rõ.

### 10. Operations & Evolution — Vận hành và cải tiến

- [ ] Health monitoring, logs, metrics, traces theo nhu cầu chẩn đoán.
- [ ] SLI/SLO và alerting ở mức phù hợp; tránh alert noise.
- [ ] Incident response, severity, escalation, postmortem/root cause.
- [ ] Patching, vulnerability management, secret rotation, access reviews.
- [ ] Backup verification và restore drills theo độ quan trọng dữ liệu.
- [ ] Capacity/performance tuning theo **đo đạc**, không tối ưu cảm tính.
- [ ] Cost tracking, budgets, unit economics và quyền sử dụng dịch vụ.
- [ ] Product analytics, user feedback, prioritized improvements.
- [ ] Technical debt được ghi nhận, đánh giá tác động và xử lý theo ưu tiên.
- [ ] Recurring compliance/privacy/data-retention checks khi áp dụng.

**Deliverables:** Dashboards, runbooks, incident records, maintenance backlog, cost/SLO review.  
**Gate 10 — PASS định kỳ khi:** Dịch vụ đáp ứng mục tiêu vận hành, rủi ro và chi phí được kiểm soát; vấn đề có owner và hành động tiếp theo.

### 11. Retirement — Ngừng vận hành

- [ ] Quyết định ngừng dịch vụ, business impact và exit plan.
- [ ] Thông báo cho users, partners, stakeholders theo nghĩa vụ.
- [ ] Kế hoạch export/migrate dữ liệu và kiểm tra tính toàn vẹn.
- [ ] Xử lý retention, legal holds và secure deletion theo quy định áp dụng.
- [ ] Thu hồi tokens, secrets, API keys, accounts, service access.
- [ ] Ngừng jobs, webhooks, integrations, DNS/infra và billing đúng thứ tự.
- [ ] Lưu tài liệu/bằng chứng cần thiết, xác minh không còn dữ liệu/dịch vụ mồ côi.
- [ ] Final security, financial và compliance closure.

**Deliverables:** Decommission plan, data disposition evidence, closure report.  
**Gate 11 — PASS khi:** Không còn rò rỉ dữ liệu, quyền truy cập, chi phí hoặc phụ thuộc vận hành chưa xử lý.

---

## C. 8 trục kiểm soát xuyên suốt

| Trục | Kiểm soát tối thiểu | Bằng chứng điển hình |
|---|---|---|
| **1. Governance** | Ownership, decisions, risk/change management, approvals | ADR, risk log, change record |
| **2. Security** | Secure SDLC, least privilege, threat model, secrets, supply chain | Security checks, dependency audit |
| **3. Quality** | Requirements traceability, correctness, tests, independent review | Acceptance criteria, test evidence |
| **4. Reliability** | Failure handling, resilience, backup/restore, rollback | Recovery test, runbooks |
| **5. Data** | Integrity, privacy, lifecycle, retention, migration | Schema/migration tests, data policy |
| **6. Observability** | Actionable logs, metrics, traces, alerting | Dashboards, alert rules |
| **7. Cost** | Budget, resource efficiency, capacity, operational/AI spend | Cost report, budgets/limits |
| **8. Documentation** | Contracts, decisions, operational knowledge, traceability | PRD, ADR, README, runbooks |

**Áp dụng:** Các trục trên không cần biến thành 8 team, 8 framework hoặc 8 bộ tài liệu độc lập. Tích hợp vào từng task, review và release gate.

## D. Module chỉ bật khi có nhu cầu thực tế

| Điều kiện kích hoạt | Hạng mục bổ sung | Không mặc định áp dụng |
|---|---|---|
| Xử lý bất đồng bộ, event delivery | Queue/jobs, timeout, retry/backoff, idempotency, DLQ, deduplication | Microservices, Kafka, distributed locks |
| Nhiều phiên bản app cùng truy cập DB | Expand-and-contract migration, compatibility tests, rollback/data recovery | Zero-downtime choreography cho app đơn instance ít rủi ro |
| Public APIs, payments, quota abuse | Rate limiting, anti-abuse, idempotency keys, audit trail | Distributed coordination khi DB constraint là đủ |
| High availability / traffic lớn | Load tests, SLO, graceful degradation, capacity planning, canary | Multi-region, auto-scaling phức tạp trước khi có số liệu |
| PII, dữ liệu tài chính/y tế, compliance | Privacy impact review, retention, encryption, access audit, masked test data | Sao chép production data thẳng sang staging |
| **AI/LLM-native** | Prompt/model versioning, eval datasets, regression evals, safety/guardrails, human review khi cần, fallback, latency & token-cost budgets, logging không lộ dữ liệu nhạy cảm | Vector DB, multi-agent, fine-tuning hoặc fallback chain khi chưa có nhu cầu |

**Lưu ý AI:** Quality gate bổ sung cần kiểm tra **chất lượng đầu ra trên bộ eval đại diện**, an toàn, độ ổn định, latency và chi phí mỗi tác vụ — không chỉ kiểm tra API trả HTTP 200.

## E. Quality Gate — mẫu ghi nhận dùng lại

```md
### Gate: [Phase / Feature / Release]
- Owner: ...
- Scope / Version: ...
- Acceptance criteria: ...
- Evidence (test run, diff, dashboard, document): ...
- Security / privacy / data risk: ...
- Open blockers: ...
- Conditional N/A + rationale: ...
- Decision: PASS | FAIL | CONDITIONAL (có owner, mitigation, expiry)
- Decision date: ...
```

**Quy tắc release:** Không cho phép `CONDITIONAL` với blocker nghiêm trọng về an toàn, mất dữ liệu, lộ thông tin, sai nghiệp vụ trọng yếu hoặc vi phạm pháp lý. Không ghi PASS nếu thiếu bằng chứng cho acceptance criteria bắt buộc.

## F. Tối thiểu hóa artifact để tránh bureaucracy

Đối với đa số team nhỏ–vừa, có thể bắt đầu bằng **6 nhóm tài liệu** (gộp nhiều mục trong một file nếu tiện):

1. **PRODUCT / REQUIREMENTS:** Problem, users, scope, NFR, acceptance criteria.
2. **ARCHITECTURE:** Components, APIs, data, security, key ADR.
3. **PLAN / BACKLOG:** Milestones, tasks, owners, dependencies.
4. **TEST / EVIDENCE:** Automated checks, QA, UAT records.
5. **RELEASE / OPERATIONS:** CI/CD, deployments, rollback, runbooks, alerts.
6. **DECISIONS / RISKS:** Trade-offs, exceptions, risk acceptance, closure.

Không yêu cầu tách file theo từng giai đoạn. Mục tiêu là **đủ thông tin để xây, kiểm chứng, phát hành, vận hành và bàn giao một cách đáng tin cậy**.

Đây là **6 vai trò thông tin**, không phải 6 file bắt buộc. Giữ một source of truth cho mỗi vai trò; CI/tracker/release record là nguồn bằng chứng hợp lệ. Xem [REPOSITORY.md](REPOSITORY.md) về ownership/naming và [TASKS.md](TASKS.md) về task/effort. **ECC sở hữu HOW**, không sao chép skills/rules/workflows từ ECC vào repo này.

## G. Nguồn tham chiếu chính thức

- [ISO/IEC/IEEE 12207:2026 — Software life cycle processes](https://www.iso.org/standard/90219.html): Khung quá trình vòng đời phần mềm; **không quy định đúng 11 giai đoạn**.
- [ISO/IEC 25010:2023 — Product quality model](https://www.iso.org/standard/78176.html): Mô hình đặc tính chất lượng phần mềm.
- [ISO/IEC/IEEE 29148:2018 — Requirements engineering](https://www.iso.org/standard/72089.html): Kỹ thuật quản lý và đặc tả yêu cầu.
- [NIST SP 800-218 — Secure Software Development Framework](https://csrc.nist.gov/pubs/sp/800/218/final): Thực hành bảo mật trong SDLC.
- [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/): Hướng dẫn accessibility cho web.

> **Nguyên tắc cuối:** Checklist phải phục vụ việc giao sản phẩm đúng, an toàn và có thể duy trì; không phải chạy theo điểm số “10/10” hoặc triển khai các công nghệ chưa cần thiết.
