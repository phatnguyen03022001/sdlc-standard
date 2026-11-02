# Task Contract — ChatGPT Architect → ChatGPT/Codex Executor

**Mục tiêu:** hoàn thành một outcome có thể chứng minh, với context và chi phí tối thiểu nhưng **không giảm correctness, security, privacy hoặc evidence**. [SDLC.md](SDLC.md) sở hữu lifecycle/gates; [REPOSITORY.md](REPOSITORY.md) sở hữu tài liệu/authority của source. Đây là **hướng dẫn Markdown**, không phải router, permission system hoặc runtime đang chạy.

## 1. Vai trò và lựa chọn executor

- **Architect (ChatGPT):** đọc facts hiện hành của target, xác định intent/root cause, chọn nhỏ nhất đúng, chất vấn security/data/compatibility, quyết định scope, acceptance, rủi ro, điểm dừng và task handoff. Không tự chế business rules hoặc dùng trí nhớ chat làm nguồn sự thật.
- **Executor (ChatGPT hoặc Codex):** chỉ thực thi trong phạm vi được phép; đọc đúng target/files/skills, implement, verify và báo bằng chứng. ChatGPT chỉ là executor thực sự **nếu phiên đang có công cụ cần thiết**; không có file/terminal/write tool thì chỉ có thể thiết kế hoặc hướng dẫn, **không được khai đã chỉnh repo**.
- **Reviewer độc lập:** phiên/actor thực sự khác, chỉ kích hoạt khi risk, policy hoặc owner yêu cầu. Self-review/diff check của executor **không phải** independent review.
- **Một phiên có thể thiết kế → sửa → kiểm chứng** với task nhỏ đủ rõ. Architect → Executor là mô hình ưu tiên cho handoff có chủ đích, **không phải hai agent bắt buộc**.

**Routing theo công việc, không hardcode model:** XS/S rõ contract → executor trực tiếp; M → vertical slice với acceptance; L/XL hoặc nhiều bất định → Architect investigation/read-only, chốt boundary rồi chia slice khi có lợi. Tăng reasoning effort/chọn model mạnh hơn **chỉ khi** chất lượng đo được hoặc rủi ro cần; trước đó sửa context, tools và task scope. So sánh **chi phí hoàn thành đã verify** = tokens + retries + review + thời gian + runtime, không chỉ giá token. Không hứa đạt một “giới hạn trần” tuyệt đối: đo quality/latency/cost trên task thực.

### Định tuyến capability (logic trong Markdown, không tạo router/runtime)

| Intent từ yêu cầu hiện tại | Cách xử lý | Gate không được bỏ |
| --- | --- | --- |
| **READ_ONLY** — tra cứu/audit | Researcher/Architect đọc và báo facts, không sửa | Provenance, freshness, nguồn và unknowns |
| **DESIGN** — lựa chọn material, yêu cầu mơ hồ | ChatGPT Architect xác minh requirements, đánh giá phương án và chốt boundary | Material-design-readiness trước khi code phụ thuộc quyết định |
| **IMPLEMENT** — bug/feature đã rõ | ChatGPT có tool phù hợp **hoặc** Codex thực thi | Scoped authority, focused + affected regression |
| **REVIEW** — đánh giá change/evidence | Tự review là kiểm nội bộ; reviewer khác chỉ khi cần độc lập | Đúng candidate, contract, evidence, independence không giả |
| **SECURITY** — auth/secrets/trust/privacy | Bổ sung threat-path review + test theo rủi ro | Không tự cấp quyền probe dữ liệu/live systems |
| **RELEASE** — phát hành/migrate | Tách khỏi DONE, chỉ thực hiện khi được cấp quyền riêng | Approval, rollback/recovery, smoke + quan sát |

Một task có thể đi qua nhiều capability **tuần tự trong cùng phiên**; không tự tạo agent, hội đồng hoặc song song writer. DESIGN chỉ bắt buộc cho quyết định material chưa rõ, không chặn bug nhỏ đã có contract. Router chọn **cách làm**, không chọn quyền, scope hoặc người phê duyệt.

## 2. Handoff tối thiểu

Giao **một outcome**, không giao “fix all” hoặc danh sách thay đổi tùy tiện. Có thể dẫn link đến spec/issue thay vì lặp lại:

~~~md
Outcome: [hành vi hoặc kết quả đo được]
Target: [owner/repo, branch, baseline commit, local root nếu thực thi]
Scope / non-goals: [boundary, các phần không được đổi]
Authority: [READ | EDIT | COMMIT | PUSH | RELEASE; quyền nào được operator cấp riêng]
Governance: [nguồn áp dụng + revision/content identity khi cần cross-session binding]
Material decision: [target owner/reference + RESOLVED | UNRESOLVED | N/A, khi material]
Acceptance: [testable business/contract invariants + failure paths]
Verify: [target-native commands/checks, affected consumers, expected evidence]
Risks: [security/data/compatibility, rollback khi liên quan]
ECC HOW: [skill cần dùng + source/revision đã kiểm chứng, hoặc N/A]
Stop/escalate: [blocker thực sự, điều kiện dừng]
~~~

Không bắt buộc metadata `Governance`/`Material decision` cho task thường đã rõ, không cần binding qua phiên hoặc quyết định material. Khi cần, xác minh governing source thật sự áp dụng cho target (reference repo không tự thành authority), và pin nội dung có thể truy xuất: commit SHA chỉ định danh phần đã commit, còn thay đổi local phải có content/diff identity riêng; nguồn stale/không truy xuất được → rebind hoặc NOT VERIFIED. `UNRESOLVED` không được ghi thành `N/A` để vượt gate. Không cần tạo file task/report cho handoff: gửi trong chat hoặc cập nhật **tracker hiện có**. Viết acceptance quan sát được: **điều kiện → thao tác → kết quả + điều không được xảy ra → cách chứng minh**. Khi chưa biết nguyên nhân, investigation có giới hạn trước mutation. Giao một vertical slice kiểm được; không chia FE/BE/DB theo vai trò nếu một thay đổi nguyên tử có thể hoàn tất an toàn. Nếu quyết định đổi architecture/business scope là material, Architect phải đánh giá lại trước khi executor mở rộng phạm vi.

### Material-design-readiness — chỉ bật khi code phụ thuộc quyết định quan trọng

Trước khi executor triển khai thay đổi **material** (architecture/stack, build-vs-buy, public API, data boundary, auth, migration, third-party, chi phí hoặc compatibility), Architect phải xác minh:

1. **Target truth:** owner của hành vi, AC, contracts, invariants, affected consumers và boundary FE/BE/data (khi liên quan); research/chưa chốt không tự thành requirement.
2. **Decision:** phương án đơn giản đáp ứng NFR, ít nhất một alternative có ý nghĩa, trade-off gồm security/latency/operations/cost/exit, giả định và trigger thay đổi.
3. **Proof:** bằng chứng kỹ thuật đáng tin (phiên bản, nguồn official, ngày kiểm chứng hoặc đo thực) cho lựa chọn mà code sắp phụ thuộc; có acceptance + regression + rollback/migration khi cần.
4. **Disposition:** lưu quyết định bền vững vào **owner tài liệu sẵn có của target** (hoặc tracker/ADR hiện hữu), dẫn link/trạng thái trong handoff; material decision còn `UNRESOLVED` thì **chặn dependent implementation**, chỉ giao **read-only investigation** có giới hạn để giải quyết. Chỉ dùng `N/A` khi không có quyết định material liên quan; không tạo tài liệu/proposal mới chỉ để qua gate.

Công việc cơ học, đảo ngược dễ và đã có contract rõ **không cần** extra design gate. Nếu lựa chọn đòi hỏi mua dịch vụ/cấp quyền/bẻ compatibility hay thay đổi scope đã duyệt, không coi quyết định của Architect là giấy phép để executor tự làm.

## 3. Permission & safe execution — rút gọn từ Foundation

**Authority** (được phép làm), **capability** (công cụ/môi trường hiện thực hiện được) và **evidence** (kết quả chứng minh acceptance) là ba điều kiện độc lập, phải cùng thỏa; quyền áp dụng theo [REPOSITORY.md](REPOSITORY.md). Chỉ đọc mặc định khi chưa có quyền ghi. Không tìm cách né tool/platform security block.

**Preflight trước consequential mutation:** xác minh các capability bắt buộc cho execution **và verification của giai đoạn hiện tại**, kể cả khả năng chạy checks mà acceptance thực sự yêu cầu. Thiếu capability → chọn surface tương đương đã được phép, chứng minh cùng evidence contract; nếu không có thì **BLOCKED trước mutation**. Không bắt preflight deploy/release cho task chỉ sửa code/tài liệu hoặc phases chưa tới.

| Hành động | Gate trước khi làm |
| --- | --- |
| Đọc/nghiên cứu | Bind repo và nguồn phù hợp; không suy diễn từ cwd/chat memory |
| Sửa file | Yêu cầu hiện tại cấp quyền sửa đúng scope; kiểm HEAD, worktree, index, untracked/ignored/in-flight Git; giữ nguyên thay đổi của operator |
| Commit | **Quyền commit rõ ràng**, chỉ stage paths đúng task, xem diff/checks |
| Push | **Quyền push rõ ràng**, xác minh remote ref mới, fast-forward/ancestry an toàn và xác nhận kết quả published |
| Deploy/release/destructive/force | **Quyền riêng cho thao tác cụ thể** cùng safety, rollback, approvals trong target |

### Execution surface — ChatGPT và Codex

Codex ưu tiên filesystem, terminal/Git CLI và verification commands native khi có. ChatGPT dùng native hoặc connected execution tools thực sự có trong phiên và được cấp quyền; không giả định có filesystem/terminal. **MCP không bắt buộc**; không cài/configure MCP hoặc thêm abstraction chỉ để thực thi. Transport không phải architecture authority; capability sẵn có không cấp quyền.

### Git synchronization — canonical execution contract

**GitHub-first, local execution:** GitHub là canonical **published** source; local chứa execution state, uncommitted/unpushed work và local-only configuration. SHA bằng nhau không chứng minh working content giống nhau. Một authorized writer trên mỗi vùng có thể giao nhau.

**Task-boundary preflight:** trước consequential repository work, trước commit/publication, sau publication và trước khi khai `SYNCED_CLEAN`: bind actual root, canonical origin (fetch/push), branch/HEAD và applicable instructions → kiểm index/worktree/untracked/ignored liên quan, in-flight Git operation → fresh-query remote branch bằng `git ls-remote origin refs/heads/<branch>` hoặc công cụ tương đương → phân loại bằng exact commits và ancestry. Local tracking refs không thay fresh remote observation. Thiếu capability, authority cần thiết, remote ref không xác minh được hoặc Git operation chưa giải quyết → **BLOCKED**; không mutation để đoán trạng thái.

Khi thiếu commit object để kiểm ancestry, chỉ fetch đúng origin/ref khi được phép, rồi đối chiếu lại fresh remote; không blind pull/push/retry. Không tự reset, clean, rebase, stash, merge commit, force-push, thay history hay tạo duplicate clone/worktree để đồng bộ.

| Fresh remote so với local HEAD | Điều kiện / hành động |
| --- | --- |
| **Equal** | Không có in-flight operation, worktree/index sạch → **READY**. Dirty → inspect exact diff, xác định ownership/accepted scope, bảo toàn thay đổi; chưa phải published content. |
| **Remote ahead** — local là ancestor của remote | Chỉ khi branch fast-forward được, worktree/index sạch, không có untracked/ignored path collision với incoming changes và có quyền pull: `git pull --ff-only origin <branch>`. Kiểm actual HEAD, fresh remote và clean state; reload governing contracts đã đổi, revalidate authority/acceptance và evidence trước khi tiếp tục. |
| **Remote ahead + dirty index/worktree hoặc path collision** | Không pull, không stash/reset/rebase hay tạo commit chỉ để dọn tree. Inspect ownership, acceptance, publication authority và overlap; không chứng minh bảo toàn/reconcile an toàn → **BLOCKED**. |
| **Local ahead** — remote là ancestor của local | Chỉ publish khi **mọi commit chưa published** và toàn bộ candidate đúng accepted/authorized scope, required checks đạt, có quyền commit/push tương ứng. Commit hợp lệ nhưng chưa có quyền push → giữ local, không publication. |
| **Diverged** — không bên nào là ancestor của bên kia | **BLOCKED**; báo exact local/remote commits và divergence, phương án reconciliation cần authority bổ sung; không tự merge/rebase/reset/force-push/recreate. |
| **Unknown publication outcome** | Giữ original operation identity khi có; fresh-query remote, kiểm expected commit và ancestry, rồi dùng **Unknown-effect recovery** bên dưới. Expected commit đã nằm trong remote history → effect present; remote HEAD khác expected vẫn chưa `SYNCED_CLEAN`. Không suy timeout là thất bại hoặc stdout thành công là remote proof. |

**Candidate → publication:** kiểm exact full candidate theo mục 5, accepted scope và fresh remote relationship trước commit → stage explicit authorized paths (không blind `git add .`/`git add -A`) → inspect staged diff/content identity và checks → authorized commit → kiểm resulting commit/content, worktree và fresh remote ngay trước push → chỉ push đúng origin/branch khi fast-forward-safe → requery remote để xác nhận expected commit. Candidate/content hoặc material inputs đổi thì reverify phần bị ảnh hưởng. Local commits/unrelated changes không được cấp quyền không tự thành accepted; exclude an toàn hoặc **BLOCKED**, không đè/xóa/publish chúng.

Remote đổi giữa check và push hoặc push bị rejected → **reconcile trạng thái thật trước khi quyết định tiếp**, không mặc định retry hay force. Normal non-force push bảo vệ fast-forward tại server; precheck không khóa remote. Sau pull làm đổi candidate phải reload/reverify phần bị ảnh hưởng. Nếu push mất acknowledgement, dùng original operation/postcondition và fresh remote để xác định effect; unresolved → **BLOCKED**.

**Clean completion:** dirty được phép trong execution; không ép clean bằng destructive cleanup, commit ngoài scope hoặc xóa `.env`, secrets, ignored cache/dữ liệu local-only. Kiểm collision bằng path, không in secret content. Không tạo daemon, watcher, cron, hook, background polling, sync service hay dependency mới; Markdown chỉ hướng dẫn execution trong phiên, không tự đồng bộ/enforce Git.

| Trạng thái | Bằng chứng bắt buộc |
| --- | --- |
| `VERIFIED` | Exact candidate đạt required checks; chưa chứng minh publication. |
| `COMMITTED_LOCAL` | Commit tồn tại local; chưa xác nhận GitHub. |
| `PUBLISHED` | Fresh remote xác nhận expected commit trong published history; chưa chứng minh clean/sync. |
| `SYNCED_CLEAN` | Publication confirmed, actual local HEAD **bằng fresh remote HEAD**, `git status --porcelain=v1 -uall` **không có output**, không unresolved Git operation; ignored local-only config được bảo toàn. |
| `PARTIAL` | Một phần outcome hoàn thành nhưng thiếu acceptance; nêu rõ phần thiếu. |
| `BLOCKED` | Không thể tiếp tục an toàn, thiếu authority/capability hoặc chưa reconcile được state/effect. |

Không thể clean an toàn bằng thao tác đã cấp quyền hoặc không fresh-verify được remote → `PARTIAL`/`BLOCKED`, không khai `SYNCED_CLEAN`. `VERIFIED ≠ PUBLISHED ≠ SYNCED_CLEAN`.

**Unknown-effect recovery:** Với consequential operation dài, có thể mất acknowledgement hoặc không retry an toàn, **trước dispatch** giữ original operation locator/identity nếu surface hỗ trợ, expected postcondition và cách reconciliation bằng công cụ hiện có. Không áp đặt locator lên read-only commands hoặc routine local edits có postcondition dễ kiểm tra. Sau interruption: rebind target + scoped authority → tra original operation nếu recoverable → kiểm local/external postcondition thật → phân loại `EFFECT_PRESENT` / `EFFECT_ABSENT` / `UNRESOLVED`. Chỉ retry khi chứng minh `EFFECT_ABSENT` **và** preconditions còn hợp lệ, hoặc exact operation contract chứng minh replay-safe; `UNRESOLVED` → `OUTCOME_UNKNOWN` / **BLOCKED**, không blind retry. Không cần daemon, lease hoặc store mới.

**Continuity tối thiểu khi đổi ChatGPT ↔ Codex/phiên:** chuyển handoff gốc + target/ref/HEAD hiện tại, changed paths và dirty state, acceptance đã chứng minh (lệnh + kết quả + candidate), phần chưa xong, original operation locator/effect chưa xác định, bước kế tiếp **đã được cho phép**. Người nhận phải tự đối chiếu trạng thái thực; bằng chứng chỉ tái sử dụng khi candidate, điều kiện test và công cụ còn khớp. Không biến chat transcript thành source of truth.

**Escalate thay vì đoán:** mất authority/target identity, conflicting requirements, không xác định impact, security/data risk cao, migration không rollback, unknown-effect write, thay đổi không thể verify, hoặc review thực sự độc lập bắt buộc. Report blocker rõ ràng, không “tự phê duyệt”.

## 4. ECC — tận dụng tối đa, không sao chép framework

**ECC = HOW** (investigate/design, implementation, TDD, testing, review, debugging, deployment). Target = product/engineering truth; SDLC = required outcome/gates; [REPOSITORY.md](REPOSITORY.md) = knowledge ownership. ECC **không cấp quyền**, không vượt target instructions, và skills/hook dành riêng Claude Code **không tự chạy** trong ChatGPT/Codex.

**Luồng:** Intent → Target truth → ECC discovery → Revision verification → Applicability → Selective HOW → Target verification.

1. Xác minh **ECC source/skill thực sự tồn tại ở exact commit revision** đang sử dụng; checkout workspace có thể khác revision Foundation tham chiếu, không hardcode Foundation pin hoặc suy đoán đã installed/activated. Nếu checkout có thay đổi local, commit SHA một mình không định danh nội dung thực đang đọc; đối chiếu content identity hoặc không coi là pinned.
2. Chọn **1–2 skills ban đầu** đúng intent thay vì đọc cả catalog; nạp thêm theo risk/gate thực tế, không đặt trần 2 skills cho cả task hoặc lấy số skill làm KPI.
3. Với mỗi skill thực sự dùng, xác định `name/source@revision → applicable sections → adopted/skipped guidance (lý do nếu khác biệt material) → target-native verification`; chỉ ghi đủ để truy xuất khi cần, không ép metadata cho task nhỏ đã rõ. Dùng commands/tests của target và điều chỉnh cho ChatGPT/Codex.
4. Bỏ qua hướng dẫn không tương thích với target, authority hoặc harness (Claude hooks/orchestration, global config, coverage threshold không thuộc target policy, approvals thừa, dependency/infra mới không có nhu cầu); không tự tải/cài/nâng cấp hay đánh dấu ECC PASS khi thiếu evidence.

**Map ECC theo tình huống — tra tên trong checkout thật, không tự kích hoạt:**

| Tình huống | ECC HOW tham khảo |
| --- | --- |
| Product intent/feature boundary chưa rõ | `intent-driven-development`, `product-capability` |
| Feature/change/defect thực thi | `orch-add-feature`, `orch-change-feature`, `orch-fix-defect` — chọn **một** đúng intent |
| Shared API/event/consumer thay đổi | `contract-first` |
| Code/tests và hồi quy | `tdd-workflow`, `verification-loop` — dùng phần phù hợp stack/test thật |
| Auth/trust/secrets/production risk | `security-review`, `production-audit` khi impact cần |
| Docs/knowledge drift | `living-docs-governance` |

**Priority HOW:** target-native commands/conventions → ECC skill đúng revision → bổ sung kỹ thuật khác **chỉ khi** ECC thiếu capability cần thiết, có bằng chứng và được phép. ECC HOW có thể đề xuất thêm file/tool/approval; chỉ nhận khi đáp ứng scope và admission của [REPOSITORY.md](REPOSITORY.md). Không cài hook/agent, không dùng cron, tự động hóa hay subprocess chỉ vì một ví dụ ECC.

Không thêm abstraction/dependency/infrastructure chỉ vì ECC có skill. Dùng capability cần thiết với context tối thiểu; hiệu quả được đánh giá bằng **đúng task + regression + security + cost**, không phải số lượng skill.

## 5. Verification, review, report

- **READY:** đúng target/authority, outcome/contract/acceptance, dependency và test surface rõ.
- **VERIFY:** chạy checks liên quan trên môi trường thật; private helper → unit; shared module → direct/transitive consumers; API/schema → contract/integration & compatibility; auth/payment/migration/config → security, data, failure-path và broader regression theo rủi ro. Dynamic routing/unknown consumers → mở rộng checks hoặc báo không đủ bằng chứng. Không bắt buộc full E2E sau mọi sửa.
- **DONE:** review diff/scope, acceptance đủ evidence, không còn blocker nghiêm trọng; **tests green ≠ đúng nghiệp vụ**.
- **RELEASE:** gate riêng của [SDLC.md](SDLC.md), không suy ra từ DONE.

**Candidate/evidence binding:** Candidate clean, đã commit chỉ dùng exact commit SHA sau khi kiểm working content khớp; candidate dirty cần base HEAD + affected paths + staged/unstaged state + relevant untracked content + diff/content identity có thể đối chiếu. Gắn mỗi verification với candidate thực kiểm, command/verifier profile, observed result, material environment/toolchain, inputs và assumptions. Đối chiếu candidate **sau** khi kiểm; content, test conditions hoặc material inputs thay đổi thì chỉ tái dùng evidence khi chứng minh còn áp dụng, nếu không phải reverify phần bị ảnh hưởng. Không có candidate identity đáng tin → **NOT VERIFIED**; không cần fingerprint store hay script mới.

**Independent review chỉ khi bắt buộc:** `Reviewer / Candidate / Evidence / Decision`. Reviewer phải độc lập thật nếu policy đòi hỏi; nhãn hoặc metadata không chứng minh independence. Đánh giá đúng candidate và evidence có thể kiểm; thiếu mandatory independent evidence → gate **không PASS**. Self-review chỉ là internal check.

Báo gọn trong chat hoặc tracker có sẵn:

~~~md
Result: PASS | FAIL | NOT VERIFIED
Target / candidate: [repo/branch, exact commit hoặc dirty candidate identity]
Changed: [files/surface và consumers]
Evidence: [candidate + commands/profile, observed result, material inputs/toolchain]
Skipped: [checks + lý do, không gán PASS]
Risks / blockers: [còn lại]
Git: [uncommitted | VERIFIED | COMMITTED_LOCAL | PUBLISHED | SYNCED_CLEAN | PARTIAL | BLOCKED; local/remote SHA + clean evidence khi publication]
~~~

**Adversarial check gọn trước khi kết luận:** (a) AC nào vẫn sai dù tests xanh? (b) tác động tới consumer/config/permission gián tiếp nào chưa kiểm? (c) state/remote/test output nào có thể stale? (d) failure, rollback, data leakage hoặc unauthorized change nào có thể xảy ra? (e) thao tác nào vẫn UNKNOWN? Chỉ mở rộng kiểm tra khi có failure mode cụ thể; không giả vờ đã có independent reviewer.

**PASS chỉ khi bằng chứng đáp ứng toàn bộ acceptance liên quan.** Independent review phải được gắn đúng nguồn; release permission tách riêng. Không sinh report Markdown, task YAML hay lịch sử chat vào repo chỉ để trông “đủ quy trình”.
