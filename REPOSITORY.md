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

## 6. Knowledge base: curation, lifecycle và feature ownership

**Mục tiêu:** một người/agent mở đúng tài liệu cần thiết trong vài lần điều hướng; không đọc cả kho history hoặc tài liệu nghiên cứu để hiểu một feature.

| Lớp | Sở hữu | Trạng thái / vị trí gợi ý |
| --- | --- | --- |
| **Canonical (normative)** | Những fact và quyết định hiện đang áp dụng cho target: spec, contracts, invariants, runbooks | Đặt ở source hiện hữu; một owner, có link từ README/index |
| **Research (informative)** | Lựa chọn và bằng chứng còn có thể thay đổi, chưa chắc áp dụng | Một thư mục `research/` **chỉ khi đã có nhu cầu tra cứu nhiều chủ đề**; xem [research/README.md](research/README.md) |
| **Evidence/history** | Test outputs, CI, incidents, decision history | CI/tracker/monitoring/Git; **không chép lại** thành report Markdown |
| **Scratch** | Chat transcript, draft, benchmark chưa kiểm chứng, tạm lưu | Ngoài repository; không commit |

**Không có quy tắc bắt buộc mỗi feature một docs file**. Khi feature đủ phức tạp mới cần một canonical feature spec; mặc định hợp nhất business rules, API contracts, acceptance, owners trong spec/issue hiện có. Nếu target có nhiều feature specs, gợi ý `docs/features/<feature-slug>.md` (một file per **domain capability**, không per component, endpoint, task hay sprint). Tài liệu feature chỉ chứa intent, business invariants, external contracts, acceptance/risks và link tới tests/code — ECC sở hữu kỹ thuật triển khai.

**Admission gate (phải đạt cả 5):**
1. Câu hỏi hoặc quyết định nào đang thiếu nguồn thật? Người đọc là ai?
2. Có thể **sửa hoặc link** nguồn cũ thay vì tạo bản mới không?
3. Fact hay nhận định? Nguồn gốc + ngày kiểm chứng + mức bất định?
4. Owner chịu trách nhiệm; khi code/contract thay đổi, tài liệu nào phải sửa?
5. Nếu research: trigger tái kiểm chứng và hành động `PROMOTE → UPDATE → DELETE` là gì?

**Sau khi có câu trả lời:** promote kết luận được chấp nhận vào canonical owner/ADR, giữ link nguồn cần thiết; remove bản nghiên cứu không còn giá trị, Git giữ lịch sử. Không tạo `archive/`, `deprecated/` hay `v2-final/` chỉ để tích trữ. Tài liệu nào **không còn chủ sở hữu, không còn đọc hoặc chỉ trùng nội dung code/tests** thì gộp/xóa trong cùng change; không dựa duy nhất vào ngày cũ để xóa khi nội dung vẫn còn chính xác.

**Change-impact check:** Khi thay đổi feature, kiểm tra spec/contract/ADRs/runbook nào có assertion bị ảnh hưởng; cập nhật **cùng commit** nếu có drift. Status và các câu chưa giải quyết thuộc **một tracker**, không duplicate bảng status ở Markdown. Chỉ kiểm audit/link theo scope thay đổi; không cần hệ thống docs governance chạy nền.

**Boundary với ECC docs-governance:** ECC có artifact-role contract, kỹ thuật audit và router. Standard này chỉ quy định **tiêu chí vào/ra của tri thức**, không cài lại ECC scripts, map/status/history files hay tự nhân bản `CONTEXT.md`/`TESTS.md`.

## 7. Code ownership, shared reuse và consumer contracts

**Áp dụng ở target repo, không áp đặt cấu trúc thư mục:** Ưu tiên colocate code trong feature sở hữu hành vi. Chỉ đưa logic vào shared khi các consumers thật sự dùng **cùng semantics/invariants**, có contract rõ và việc dùng chung giảm chi phí thay đổi; hai đoạn code giống nhau hoặc dự đoán sẽ tái sử dụng chưa đủ để extract. Với nhiều consumer nhưng khác business rules, giữ riêng.

- **Owner:** Mỗi shared module có một owner logic (feature/platform phù hợp), contract đầu vào/đầu ra, lỗi và các invariants cần giữ. Tránh `utils`/shared "god module", circular dependencies, hoặc chia sẻ internals giữa các feature.
- **API tối thiểu:** Chỉ export phần consumers cần. Giữ boundary ổn định; không thêm generic abstraction, interface, package hay dependency chỉ để "future-proof". Shared có một consumer vẫn hợp lệ khi mang tính nền tảng thật sự, không lập luật cứng "phải có ≥2 consumers".
- **Trước khi thay đổi shared:** Xác định direct/transitive consumers, cả các đường gọi gián tiếp (config, DI, runtime routing), và hành vi/contract bị ảnh hưởng. Nếu không xác định đủ dependency, **mở rộng phạm vi kiểm thử** thay vì mặc định an toàn.
- **Sau khi thay đổi:** Kiểm tra contract + consumer regression theo [SDLC.md](SDLC.md); ghi bằng chứng và rủi ro còn lại theo [TASKS.md](TASKS.md). Nếu module không còn owner/consumer, đơn giản hóa hoặc đưa về feature khi phù hợp.

Đây là tiêu chí **ownership và outcome**; ECC/target tooling sở hữu phương pháp refactor, dependency analysis và chạy test thực tế.
