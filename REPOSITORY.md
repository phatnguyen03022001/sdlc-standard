# Repository Contract — Docs, Rules, Structure & Naming

> Bộ nguyên tắc **áp dụng có điều kiện**, không tự sinh cây thư mục cho target repository.

## 1. Boundary (không overlap)

| Chủ thể | Sở hữu | Không sở hữu |
| --- | --- | --- |
| **Target repo** | Product requirements, business truth, code, tests, architecture decisions, runtime evidence, release | Nội dung giả định từ repo khác |
| **SDLC Standard** | Outcome theo phase, acceptance/evidence/gates, điều kiện kiểm soát | Coding style, hooks, skills, executor, task automation |
| **ECC** | HOW: coding patterns, tests, review, debugging, deployment methods, agent skills/rules | Product scope, stakeholder acceptance, quyền thực thi |
| **Agent Foundation (nếu có)** | Capability router, authority, phạm vi tác động, execution transport | Bản sao SDLC/ECC/target state |

ECC có sẵn `rules/` và `skills/` cho coding/testing/review/architecture. **Không copy, sửa hoặc phát hành lại các rule packs đó** trong repo này. Target repo có thể tham chiếu ECC phù hợp với stack đã cài; không giả định ECC đã được kích hoạt.

## 2. Docs tối thiểu là logical roles, không phải số file

| Role | Nguồn sự thật nên dùng |
| --- | --- |
| Entry/run/setup | `README.md` hiện hữu |
| Scope/requirements/acceptance | PRD/spec hiện có, issue hoặc product tracker |
| Architecture/contract/decision | Schema/API/code + ADR khi có quyết định dài hạn |
| Implementation | Source code, tests, framework conventions |
| Backlog/owner/status | **Một** issue/tracker; không tạo task ledger song song |
| Verification/UAT | Tests, CI runs, staging evidence, release record |
| Operations/recovery | Runtime config, runbooks/dashboard khi có nhu cầu |

Gộp vai trò vào một tài liệu nếu dễ maintain; không bắt buộc tạo `docs/`, `tasks/`, `reports/`, `plans/`, `decisions/`. Giữ convention đã có của repo, không ép mô hình `src/`, monorepo, microservices hay cấu trúc framework khác.

## 3. Rule tạo tài liệu mới (tất cả điều kiện phải đạt)

- Chưa tồn tại owner/source of truth có thể cập nhật.
- Nội dung cần được duy trì **sau khi task kết thúc**, có độc giả cụ thể.
- Không thể đặt gọn vào docs/tracker/code/tests hiện có.
- Có owner, tên rõ, nơi index/link đến và sự kiện phải cập nhật.

Nếu không đạt, dùng issue/PR/commit/CI/artifact store sẵn có. **Không tạo placeholder, báo cáo từng phase, duplicate spec, transcript, dump, cache, screenshot, generated scaffolds** chỉ để nhìn đầy đủ. Chỉ commit generated artifact nếu nó thực sự là sản phẩm có chủ đích.

## 4. Naming & cấu trúc

- Với repo nhỏ: giữ file ở root; chỉ lập thư mục khi giảm độ phức tạp thực.
- Tên đặc biệt: `README.md`, `AGENTS.md` (chỉ khi harness thật sự đọc và có target facts), tên bắt buộc của tool/framework.
- Tài liệu mới **khi chưa có convention**: `lowercase-kebab-case.md`, một tên cho một vai trò; tránh `v2-final-latest.md`, tên theo ngày/phase/task nếu chỉ là snapshot.
- Trong repo **này**: `README.md`, `SDLC.md`, `REPOSITORY.md`, `TASKS.md` là tên canonical.
- `AGENTS.md` (nếu cần) chỉ làm context pointer ngắn tới nguồn thật + lệnh target; **không nhúng lại SDLC, ECC hay catalog model**.
- Một owner cho một fact; tài liệu khác **link**, không copy. Khi đổi owner/path, cập nhật links và xóa bản cũ trong cùng change.

## 5. Chống rác và drift

**Trước khi tạo:** search file/tracker cũ → xác nhận ownership → lựa chọn update hoặc lý do tạo mới.  
**Trong thay đổi:** chỉnh đúng thứ bị ảnh hưởng, không refactor/rename “nhân tiện”.  
**Kết thúc task:** chỉ update docs nếu requirements, behavior, contracts, decisions, operation hoặc instructions đã đổi.  
**Trong review:** file trùng, mồ côi, stale, không owner hoặc tạo để báo cáo tình trạng nhất thời → gộp/xóa, giữ lịch sử trong Git.

Không áp đặt “đúng X file” cho mọi target; chuẩn là **ít nguồn sự thật nhất nhưng vẫn đủ correctness, security, vận hành, traceability**.
