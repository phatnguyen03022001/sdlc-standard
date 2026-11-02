# SDLC — 11 gates, 8 trục, áp dụng theo rủi ro

> **Mini v2.1 · 2026-11-02.** Chuẩn WHAT / WHEN / DONE cho web/AI-native app. Nhóm công việc theo SDLC, **không phải waterfall** hay tuyên bố chứng nhận ISO. Có thể chạy song song/lặp theo feature/release; N/A phải có lý do. [TASKS.md](TASKS.md) quản lý Architect → Executor, quyền thực thi và bằng chứng.

## 1. Vòng đời: đầu ra tối thiểu và điều kiện qua gate

| Giai đoạn | Outcome / evidence tối thiểu | Gate PASS khi |
| --- | --- | --- |
| **1. Discovery** | Vấn đề, users, value, feasibility/risk, KPI, owner, go/no-go | Bài toán và thước đo thành công đủ rõ để quyết định tiếp |
| **2. Requirements** | Functional/NFR, business rules, roles, privacy, data ownership/retention, failure paths, testable acceptance | Scope và invariants rõ, conflict quan trọng đã được giải quyết |
| **3. Architecture** | Context, boundaries, API/data contracts, stack trade-offs, security, scale/cost assumptions, ADR *khi cần* | Phương án **đơn giản nhất đáp ứng NFR**; rủi ro, compatibility và migration có hướng xử lý |
| **4. Planning** | Backlog ưu tiên, task vertical slice, owner, dependencies, budget, DoR/DoD, rollback owner | Có outcome, acceptance, thứ tự và blocker management |
| **5. Design** | Critical user flows, responsive/a11y, loading/empty/error/denied states; prototype *khi cần* | Hành vi UX chính, edge cases và handoff kiểm được |
| **6. Implementation** | Target-native code/contracts, migrations/tests, secrets/config controls, diff | Chạy build/static checks/tests phù hợp; security, data integrity và contract có evidence |
| **7. Testing & QA** | Unit/contract/integration/regression, critical E2E, security/performance/a11y *theo rủi ro* | Critical paths và affected consumers đã verify; không có unresolved blocker |
| **8. Acceptance & Readiness** | Staging/preprod validation, UAT, release decision/sign-off, migration/backup/rollback/runbook | Phê duyệt bắt buộc đã có **trước** production; không còn blocker an toàn/dữ liệu |
| **9. Deployment & Release** | Authorized deploy, compatibility-safe migration, smoke tests, logs/metrics, rollback capability | Đã xác minh production ổn theo criteria; publication và release có evidence riêng |
| **10. Operations & Evolution** | SLO/alerts, incidents/root cause, patching, backups/restore, capacity/cost, feedback | Định kỳ đạt mục tiêu vận hành; rủi ro/cost có owner |
| **11. Retirement** | Exit/migration/export, retention/deletion, revoke access, stop integrations/billing | Không còn tài nguyên, dữ liệu, quyền hoặc nghĩa vụ bị bỏ sót |

### Architecture decision gate (khi có thay đổi material)

**Không viết code phụ thuộc một lựa chọn chưa chốt.** Với thay đổi material về stack/third-party, interface, persistent data hoặc security boundary: xác minh target-owned requirements → so sánh ít nhất một phương án đơn giản hơn → đánh giá trade-off security/performance/maintainability/operations/cost → chọn với bằng chứng phiên bản/nguồn/đo → ghi ở owner hiện hữu → mới giao implementation. Mọi unknown có thể làm thay đổi boundary hoặc gây mất dữ liệu phải được điều tra có giới hạn hoặc chặn phần triển khai phụ thuộc; chi tiết chỉ mang tính implementation và thay đổi cơ học không cần ceremony này. [TASKS.md](TASKS.md) sở hữu handoff và material-design-readiness.

**Gate không đồng nghĩa file:** Đầu ra có thể là spec hiện hữu, code/tests, issue, CI run, dashboard hoặc release record. **Không** tạo 11 tập tài liệu, status ledger hoặc report files cho mỗi feature. Chỉ hoạt động/gate có liên quan mới áp dụng trong một task, nhưng không được bỏ qua critical controls.

## 2. Tám trục xuyên suốt

| Trục | Bảo đảm cần kiểm | Evidence điển hình |
| --- | --- | --- |
| Governance | Owner, decision, change/approval, scope | Target spec, ADR, issue |
| Security | Least privilege, threat/supply-chain checks, secrets | Authz tests, dependency scan |
| Quality | Traceability, behavior, regression, independent review khi bắt buộc | AC, unit/integration/E2E |
| Reliability | Timeout/failure handling, backup/restore, rollback | Failure-path/recovery checks |
| Data | Integrity, privacy, migrations, retention | DB constraints, migration tests |
| Observability | Actionable logs/metrics/traces, alerts | Runtime dashboards |
| Cost | Measured resource/AI spend, budgets, capacity | Cost and latency metrics |
| Documentation | Một source of truth, compatible contracts, runbook | Canonical docs + Git evidence |

Các trục không phải 8 nhóm tài liệu hay 8 vai trò agent. Acceptance của task chọn điều kiện áp dụng; nguy cơ thực tế quyết định độ sâu.

## 3. Gate verification theo thay đổi

| Blast radius | Mức kiểm tra tối thiểu |
| --- | --- |
| Private helper / isolated change | Focused unit/regression và static checks |
| Shared module / reusable logic | Shared contract + unit và **direct/transitive consumers** |
| Public API / schema / event | Contract/integration, compatibility across active versions |
| AuthZ / payment / persistence / migration / runtime config | Security/data/failure cases + critical integration/E2E, backup/rollback khi liên quan |
| Cross-cutting/unknown dynamic consumers | Broader suites hoặc ghi **NOT VERIFIED** nếu coverage không chứng minh được |

Ba mức kiểm: **fast feedback** (focused) → **DONE** (affected consumers, gates) → **release** (broader CI/UAT/smoke/monitoring). Không full E2E mỗi edit, nhưng cũng không tự suy đoán dependency graph đầy đủ. No actual test/evidence = NOT VERIFIED; test PASS không thay stakeholder acceptance.

## 4. Rủi ro cần bật thêm (không bật mặc định)

- **Async / external side effects:** bounded retry, timeout, idempotency, deduplication, queue/DLQ *khi cần*. Transaction/constraint trước distributed locks.
- **Mixed app versions / schema change:** backward compatibility, expand-and-contract, migration/restore evidence khi không thể dừng đồng thời.
- **Public API / payments / abuse:** authz, limits, idempotency, audit trails phù hợp.
- **High traffic / HA:** benchmark/load-test, SLO, measured capacity; canary/blue-green/multi-region chỉ khi lợi ích chứng minh được.
- **Sensitive data/compliance:** retention, least privilege, masked/synthetic test data; không mặc định copy dữ liệu thật vào staging.
- **AI/LLM-native:** version prompt/model, eval set đại diện, correctness/safety regression, latency/token-cost, fallback/human review theo rủi ro, không log PII. **HTTP 200 không chứng minh chất lượng chấm điểm/AI.**

Chỉ đưa vào thiết kế những gì requirement và evidence chứng minh cần; không mặc định microservices, vector DB, multi-agent, k8s hay fine-tuning để “scale tương lai”. Reliability/scale phải đo.

## 5. Gate quyết định & quyền phát hành

Ghi tối thiểu vào **tracker/release record đang có**, không tạo file mới:

~~~md
Gate: [phase / feature / release]
Owner + target/version: [...]
Acceptance/evidence: [...]
Security/data/compatibility risk: [...]
Skipped/N/A + rationale: [...]
Blocker + mitigation/expiry (nếu có): [...]
Decision: PASS | FAIL | CONDITIONAL | NOT VERIFIED
~~~

- **CONDITIONAL không dùng** cho blocker nghiêm trọng về an toàn, mất dữ liệu, lộ thông tin, sai nghiệp vụ trọng yếu hoặc vi phạm pháp lý; không thể thay thiếu sign-off bắt buộc.
- **DONE ≠ COMMITTED ≠ PUSHED ≠ RELEASED.** Quyền chỉnh sửa, Git publication, deploy và thao tác phá hủy tách biệt theo [TASKS.md](TASKS.md).
- Trước production: UAT/owner sign-off, migration/rollback readiness. Sau deploy: smoke, metrics, rollback/recovery nếu vượt ngưỡng; independent review khi policy/rủi ro đòi hỏi.
- Báo trung thực **PASS / FAIL / NOT VERIFIED** cho kiểm chứng thực hiện; quyết định gate CONDITIONAL cần risk owner rõ. Không tự đánh giá “100%/10 điểm” nếu không có đo lường.

## 6. Tham chiếu chuẩn (không đồng nhất với chứng nhận)

- [ISO/IEC/IEEE 12207:2026](https://www.iso.org/standard/90219.html) — software lifecycle; **không áp đặt 11 phase**.
- [ISO/IEC 25010:2023](https://www.iso.org/standard/78176.html) — product quality model.
- [ISO/IEC/IEEE 29148:2018](https://www.iso.org/standard/72089.html) — requirements engineering.
- [NIST SP 800-218](https://csrc.nist.gov/pubs/sp/800/218/final) — secure software development.
- [WCAG 2.2](https://www.w3.org/TR/WCAG22/) — accessibility.
