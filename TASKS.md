# Task Contract — ChatGPT Architect → ChatGPT/Codex Executor

**Mục tiêu:** hoàn thành một outcome có thể chứng minh, với context và chi phí tối thiểu nhưng **không giảm correctness, security, privacy hoặc evidence**. [SDLC.md](SDLC.md) sở hữu lifecycle/gates; [REPOSITORY.md](REPOSITORY.md) sở hữu tài liệu/authority của source. Đây là **hướng dẫn Markdown**, không phải router, permission system hoặc runtime đang chạy.

## 1. Vai trò và lựa chọn executor

- **Architect (ChatGPT):** đọc facts hiện hành của target, xác định intent/root cause, chọn nhỏ nhất đúng, chất vấn security/data/compatibility, quyết định scope, acceptance, rủi ro, điểm dừng, delivery/publication boundary và task handoff. Không tự chế business rules hoặc dùng trí nhớ chat làm nguồn sự thật.
- **Executor (ChatGPT hoặc Codex):** chỉ thực thi trong phạm vi được phép; đọc đúng target/files/skills, implement, verify và báo bằng chứng. ChatGPT chỉ là executor thực sự **nếu phiên đang có công cụ cần thiết**; không có file/terminal/write tool thì chỉ có thể thiết kế hoặc hướng dẫn, **không được khai đã chỉnh repo**.
- **Reviewer độc lập:** phiên/actor thực sự khác, chỉ kích hoạt khi risk, policy hoặc owner yêu cầu. Self-review/diff check của executor **không phải** independent review.
- **Một phiên có thể thiết kế → sửa → kiểm chứng** với task nhỏ đủ rõ. Architect → Executor là mô hình ưu tiên cho handoff có chủ đích, **không phải hai agent bắt buộc**.

**Model mặc định (không phải bằng chứng runtime):** Architect dùng GPT-6, ưu tiên `high` cho quyết định material/phức tạp; Main Executor dùng GPT-6 `high`, chịu trách nhiệm tích hợp/verification/báo cáo. Chỉ khi Agent Team được duyệt và capability thực sự hỗ trợ mới dùng GPT-6 Luna với effort `high` làm subagent. Kiểm model ID/effort thực tế nếu runtime cung cấp; prompt tự khai model không phải proof. **Không âm thầm thay model**; nếu không đáp ứng được yêu cầu bắt buộc, fallback Single Agent chỉ khi vẫn đủ evidence/authority, nếu không `BLOCKED`.

**Routing theo công việc, không hardcode surface:** XS/S rõ contract → executor trực tiếp; M → vertical slice với acceptance; L/XL hoặc nhiều bất định → Architect investigation/read-only, chốt boundary rồi chia slice khi có lợi. Tăng reasoning effort/chọn model mạnh hơn **chỉ khi** chất lượng đo được hoặc rủi ro cần; trước đó sửa context, tools và task scope. So sánh **chi phí hoàn thành đã verify** = tokens + retries + review + thời gian + runtime, không chỉ giá token. Không hứa đạt một “giới hạn trần” tuyệt đối: đo quality/latency/cost trên task thực.

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

### Target lifecycle orientation — adaptive, evidence-based

**Góc nhìn thực thi hằng ngày: Understand → Change → Verify** — không phải lifecycle mới hoặc ba gate thay thế [SDLC.md](SDLC.md).

- **Understand:** xác minh đúng target, outcome/acceptance, contracts, phạm vi, quyền và rủi ro **liên quan**; không lặp Discovery nếu product intent đã được chấp nhận.
- **Change:** thực hiện thay đổi nhỏ nhất được phép, giữ business invariants, security và compatibility; quyết định material chưa chốt thì không viết code phụ thuộc.
- **Verify:** kiểm acceptance, affected consumers, exact candidate/evidence và điều kiện hoàn tất áp dụng; tests PASS không tự chứng minh acceptance/release.

**Độ sâu theo tác động, không phải checklist cố định:**

- **MINIMAL / INCIDENT-FOCUSED:** typo, cosmetic hoặc production incident → tập trung change/incident, xác minh theo phạm vi; không khảo sát toàn lifecycle.
- **LIGHT:** bug/feature có contract rõ → target-native vertical slice, root cause, focused + affected regression.
- **MATERIAL:** architecture, security/privacy, persistent data/migration, public API, major refactor, release/operations/retirement hoặc handoff đáng kể → xác minh các quyết định/gates **thực sự ảnh hưởng** (bao gồm material-design-readiness, approval, backup/recovery, independent review khi bắt buộc); chọn **gate còn thiếu nhỏ nhất** chặn outcome, không chạy lại gates đã chứng minh xong.

Ở mọi độ sâu, authorization, security/data, compatibility và release gates **áp dụng** vẫn bắt buộc; incident khẩn cấp không tự cấp quyền deploy.

Khi **material planning hoặc handoff** cần định hướng, chỉ đọc evidence có liên quan cho năm chiều **độc lập**: (a) product maturity và accepted decisions, (b) feature implementation / verification / acceptance, (c) active workstream, (d) release readiness, (e) operational status. Production app vẫn có feature đang phát triển; code ≠ verified ≠ accepted ≠ released; deployed ≠ healthy. Không bắt báo đủ năm chiều ở mọi task. Các gate và tám trục chất lượng thuộc [SDLC.md](SDLC.md).

**Nhãn cho lifecycle claim, không phải gate verdict hoặc test status:** `CONFIRMED` = nguồn có thẩm quyền với evidence phù hợp candidate/environment; `INFERRED` = suy luận có căn cứ, nêu giới hạn; `UNKNOWN` = thiếu/stale/mâu thuẫn/không truy cập được; `N/A` = không liên quan, có lý do. `UNKNOWN` không đồng nghĩa `NOT DEPLOYED` hoặc `NOT IMPLEMENTED`. Product owner quyết định requirement/acceptance, code chứng minh implementation, tests chứng minh kiểm thử, sign-off chứng minh acceptance, deploy record chứng minh release, telemetry mới chứng minh health.

**Next action:** decision chưa chốt → điều tra/chốt trước dependent code; accepted design → vertical slice; code chưa verify → relevant verification; verified chưa accepted → acceptance; accepted chưa release → release gate; incident/defect → khắc phục trọng điểm. Material handoff có thể ghi ngắn relevant state, evidence, blocker, next gate và deferred work theo mục 2; task nhỏ không cần template.

## 2. Handoff tối thiểu

**Task nhỏ, rõ và thực hiện trong cùng phiên không cần handoff riêng.** Khi thực sự bàn giao, giao **một outcome** qua ba nhóm thông tin logic, không bắt buộc template hoặc file:

- **Goal:** target cụ thể (owner/repo, branch, local root nếu thực thi), outcome và acceptance có thể kiểm chứng.
- **Boundary:** scope, non-goals và quyền operator cấp riêng cho từng hành động (`READ | EDIT | COMMIT | PUSH | RELEASE`); capability/ECC không tự cấp quyền.
- **Proof:** cách verify, evidence cần thu thập, các failure paths liên quan và điều kiện stop/escalate.

**Session bắt buộc trong mọi handoff Architect → Executor:** khai `Session: NEW CHAT | CONTINUE CHAT` **kèm một lý do ngắn**. Ưu tiên `CONTINUE CHAT` cho cùng task, correction, task kế tiếp gắn chặt hoặc accepted context còn hữu ích và không cần independence. Ưu tiên `NEW CHAT` cho workstream không liên quan, independent review, context mâu thuẫn/không đáng tin, môi trường mới cần tách biệt hoặc không thể khôi phục continuity. **Không đổi chat chỉ vì dài**; chat mới không tự chứng minh reviewer independence. Cả hai chế độ phải rebind target/source, candidate, authority, environment/capabilities và pending effects trước consequential action; không tạo session ledger/file.

Có thể trỏ tới spec/issue và instructions đã có thay vì chép lại. **Chỉ bổ sung khi liên quan:** exact baseline/candidate; material architecture decision và owner; governing source/revision cho handoff qua phiên; security/data migration; direct/transitive affected consumers; rollback/recovery; release permission; mandatory independent review; hoặc original operation identity/postcondition khi outcome chưa rõ. Không lược bỏ thông tin critical để vừa ba nhóm; việc bàn giao không thay thế authority hay [Git synchronization](#git-synchronization--canonical-execution-contract).

**Binding và acceptance:** nếu cần pin governance qua phiên, xác minh nguồn thực sự áp dụng; commit SHA không định danh thay đổi local chưa commit, cần content/diff identity tương ứng. Nguồn stale/không kiểm được → rebind hoặc `NOT VERIFIED`; quyết định material `UNRESOLVED` không được đổi thành `N/A`. Acceptance mô tả **điều kiện → thao tác → kết quả, điều không được xảy ra → cách chứng minh**. Khi root cause chưa rõ, điều tra có giới hạn trước mutation. Giao một vertical slice đủ kiểm chứng, không mặc định chia FE/BE/DB; Architect phải xem lại quyết định material trước khi mở rộng business/architecture scope.

**Architect delivery planning cho dependent tasks:** Trước khi chia/giao implementation có phụ thuộc, Architect phải xác định kỳ vọng delivery thực sự được operator chấp nhận: `local-only`, `COMMITTED_LOCAL`, `PUBLISHED` hay `SYNCED_CLEAN`; boundary publish từng task, một integration batch được chấp nhận, hoặc boundary khác đã thống nhất; quyền `EDIT`, `COMMIT`, `PUSH` riêng theo đúng repo/branch/scope; evidence hoàn tất; gate cho task kế tiếp; và cách xử lý khi không thể publish. Architect **không cấp quyền**: thiếu operator grant/capability cần cho required outcome thì nêu gap trước khi giao task như thể có thể hoàn tất. Không mặc định commit/push mỗi edit; áp dụng [Git synchronization](#git-synchronization--canonical-execution-contract) làm execution contract duy nhất.

**GitHub-first, per-task synchronization:** Nếu operator yêu cầu đồng bộ sau **mỗi publishable implementation task**, từng handoff phải ghi trong **Goal** trạng thái giao hàng `SYNCED_CLEAN` (không chỉ sửa/verify local); **Boundary** nêu chính xác target/branch/scope cùng quyền `EDIT`, `COMMIT`, `PUSH` đã được operator cấp riêng; **Proof** yêu cầu checks gắn exact candidate, resulting commit, fresh remote/HEAD equality, clean index/worktree và không có unresolved Git operation theo mục 3. Architect phải kiểm chứng bằng chứng Git/verification thực tế, không tin nhãn `PASS` không có proof; `VERIFIED`, `COMMITTED_LOCAL` và `PUBLISHED` không thay `SYNCED_CLEAN`. Không mở dependent implementation như đã READY trước khi predecessor đạt boundary bắt buộc.

**Accepted alternatives:** Một **integration batch** chỉ được hoãn publication khi operator **chấp nhận rõ boundary trước khi dùng nó**: xác định các task trong batch, phần còn unpublished, identity/scope của candidate và cross-contract verification, task/owner thực hiện integration + final synchronization, và các dependent tasks phải chờ publication đó. Không hồi tố tự đặt batch để hợp thức hóa per-task sync bị thiếu; đổi boundary cần operator chấp nhận lại cùng authority tương ứng. `local-only`/`COMMITTED_LOCAL` hợp lệ khi phù hợp delivery đã chấp nhận; research/read-only/design không phát sinh Git publication chỉ để đủ nghi thức nếu không có persistent repository change được phép.

**Cross-task continuation:** Trước dependent implementation, đối chiếu predecessor với **required** state và evidence thực tế, kể cả publication/verification. Nếu chưa đạt, không khai completed hoặc âm thầm gộp unpublished work sang task sau; chỉ tiếp tục publication/reconciliation an toàn khi có quyền và khả thi, nếu không báo `PARTIAL`/`BLOCKED` kèm điều kiện thiếu. Giữ nguyên valid local/unrelated work, không publish unaccepted changes/commits hay bypass Git conflict. Chỉ release dependency trước `SYNCED_CLEAN` nếu accepted boundary cho phép hoặc operator phê duyệt thay đổi; không chặn unrelated read-only work.

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

### Execution environment, surface & capability

**Workload routing (preference, không phải capability guarantee):** Chat cho architecture/requirements/decisions; Work cho research/investigation nhiều nguồn; Codex native cho implement/debug/test trên repo; separate Architect/reviewer session cho acceptance độc lập khi bắt buộc; macOS có toolchain phù hợp cho Apple-native verification. Chat vẫn có thể execute khi thực sự có tool và quyền; Work/Codex có môi trường, filesystem và quyền khác nhau, **không coi hoán đổi tự động**. **Không mặc định Chat hỗ trợ native Agent Teams.** MCP không bắt buộc; không cài/configure MCP hay tạo runtime chỉ để chọn mode.

**Capability evidence:** phân biệt `DOCUMENTED` (có mô tả chính thức), `VERIFIED` (đã quan sát trong đúng phiên/môi trường), `UNSUPPORTED` (bị hạn chế rõ), `UNKNOWN` (chưa xác minh). Documentation, tên mode, prompt hoặc plugin đã cài **không chứng minh capability hoạt động**. Trước thao tác cần tool đặc thù, xác minh tool, permission và proof path đang có; nếu mất capability giữa chừng, kiểm lại effect/identity trước khi fallback hoặc retry theo [unknown effects](#git-synchronization--canonical-execution-contract).

| Môi trường thực tế | Path/fallback an toàn | Evidence/boundary và điểm dừng |
| --- | --- | --- |
| **Mac + GitHub** | Ưu tiên native execution trên đúng working copy, GitHub là published truth; dùng Chat/Work/Codex theo task và tool thực tế. | Quan sát origin/HEAD, index/worktree, toolchain, checks và fresh remote; `SYNCED_CLEAN` chỉ theo Git contract bên dưới. |
| **Chỉ GitHub, Mac không truy cập được** | Dùng authorized GitHub connector/API cho scoped source edit/publication và checks khả dụng; không giả native testing. | Fresh remote/commit + diff/checks; tối đa `PUBLISHED`, local Mac `NOT OBSERVED`; native-required AC thiếu → `PARTIAL`/`BLOCKED`. |
| **Có Mac nhưng thiếu tool/permission native** | Dùng surface được phép khác **chỉ khi** chứng minh cùng contract/acceptance; không tự cài/upgrade hoặc bỏ test bắt buộc. | Ghi tool/permission thiếu, checks chưa chạy; native-only verification không thay bằng static review → `BLOCKED` phần phụ thuộc. |
| **Local ≠ remote (dirty/ahead/diverged)** | Áp dụng **duy nhất** [Git synchronization](#git-synchronization--canonical-execution-contract); không tự pull/merge/reset hoặc publish unrelated work. | Fresh HEAD/ancestry, index/worktree, overlap; reconciliation thiếu safety evidence → `BLOCKED` sync, scoped độc lập có thể tiếp tục. |
| **Tool/mode mất capability giữa task** | Dừng thao tác phụ thuộc, kiểm effect đã xảy ra và quyền hiện hành; chỉ đổi surface khi equivalent proof và authorization còn đủ. | Unknown effect → `BLOCKED` dependent operations; không blind retry hoặc suy ra thành công từ timeout. |

### Conditional Agent Team — admission & authority (không phải orchestration HOW)

**Single Agent là mặc định.** Chỉ admit Agent Team nếu **đủ cả tám điều kiện**: (1) lợi ích delegation material; (2) Architect/Owner cho phép; (3) Main có đúng operation authority; (4) runtime đã chứng minh hỗ trợ delegation cần dùng; (5) GPT-6 Luna `high` thực sự khả dụng; (6) context/tools/permission đủ và phù hợp; (7) mỗi worker có scope/output bounded; (8) Main có thể integrate và verify kết quả. Không đạt → Single Agent nếu vẫn đáp ứng acceptance; nếu task bắt buộc team/evidence không thay thế được → `BLOCKED` trước khi spawn.

**Main Executor (GPT-6 High) là single writer**: giữ toàn bộ quyền quyết định thao tác trong operator grant, integration, candidate checks, commit/push khi được cấp quyền và final reporting. Luna High subagents chỉ làm bounded **read-only** research/review mặc định; không nested delegation, không tự write/commit/push/deploy, không thừa hưởng quyền mới từ tool/skill/parent. Shared context/filesystem **không** chứng minh security hoặc reviewer independence. Không fanout tự động; concurrency/retry phải có giới hạn theo resource/risk của task, không retry mù khi timeout/effect chưa rõ. Worker fail/timeout → Main ghi partial evidence, kiểm effect, chỉ retry khi an toàn; fallback single-agent hoặc stop theo AC.

**Evidence tối thiểu nếu thực sự dùng team:** runtime/mode, observed model ID + effort nếu exposed (nếu không: `NOT VERIFIED`), worker scope/permission, input và output identity, timeout/failure, Main integration và independently verified checks. Team review không tự thành independent acceptance; mandatory reviewer phải thực sự độc lập. Chỉ có tài liệu/pattern/installed plugin **không** phải runtime activation proof.

**Tám pattern ứng viên (chọn khi có lợi, không mặc định triển khai):** Sequential Pipeline; Specialist Routing; Parallel Investigation; Map-Reduce; Orchestrator–Workers; Maker–Checker; Adversarial Review; Evaluator–Optimizer. Năm family: Sequential, Routing, Parallel, Orchestrator–Workers, Maker–Checker; Map-Reduce/Adversarial Review là variants, Evaluator–Optimizer là extension có điều kiện. **Pattern HOW** thuộc [Engineering Skills](https://github.com/phatnguyen03022001/engineering-skills), không định nghĩa orchestration algorithm tại SDLC.

### Git synchronization — canonical execution contract

**Preconditions.** GitHub là canonical **published** source; local là execution workspace, có thể chứa uncommitted/unpushed work và local-only data. Trước hành động liên quan, xác minh actual repo root, canonical origin (fetch/push), branch/HEAD, instructions, quyền riêng cho `EDIT | COMMIT | PUSH` và sync khi cần; kiểm in-flight Git operation, index/worktree, tracked/untracked/ignored paths liên quan và material dependencies. HEAD bằng remote không chứng minh working content sạch. Fresh-query `git ls-remote origin refs/heads/<branch>` (không chỉ tracking ref) khi cần xác lập remote relationship, trước publication và khi xác nhận publication/sync; nếu không xác minh được, **chặn hành động phụ thuộc**, không tự chặn scoped local edit độc lập.

**Scoped execution.** Unrelated dirty/staged changes không tự chặn task. Chỉ tiếp tục trong authorized paths khi đã loại trừ destructive overlap, accidental staging, unauthorized mutation/publication, incoming path collision (kể cả untracked/ignored) và material dependency conflict. Path disjoint **chưa đủ** nếu thay đổi incoming ảnh hưởng semantics. Bảo toàn nguyên trạng unaccepted changes, `.env`/secrets, ignored và local-only data (không in secret content). Conflict chưa giải quyết → dừng mutation xung đột, không tự overwrite, reset, clean, stash, rebase, merge commit, force-push, rewrite history hoặc tạo duplicate clone/worktree để né kiểm tra. In-flight Git operation chưa giải quyết → chặn mutation không an toàn.

| Quan hệ dựa trên fresh remote HEAD và Git ancestry | Hành động an toàn |
| --- | --- |
| **Equal** | Inspect actual index/worktree; clean không bắt buộc cho scoped task hoặc publication. |
| **Remote ahead** | Clean local: chỉ `git pull --ff-only origin <branch>` khi có quyền đồng bộ, đã kiểm incoming paths/ignored collisions và preservation. Dirty local: có thể tiếp tục **scoped local work** sau khi kiểm overlap/dependencies; **không auto-pull**. Sync dirty chỉ khi chứng minh độc lập rằng reconciliation giữ toàn bộ dữ liệu và được phép; không chứng minh được thì chặn **sync**, không tự chặn công việc local độc lập. Sau fast-forward, reload changed contracts và reverify affected candidate/evidence. |
| **Local ahead** | Chỉ publish nếu **mọi unpublished commit** (không chỉ staged paths hoặc latest commit) đúng accepted/authorized scope, required checks và quyền push; nếu thiếu quyền push, giữ local. |
| **Diverged / missing ancestry evidence** | Không tự merge/rebase/reset/force; fetch đúng origin/ref khi được phép để kiểm native ancestry. Thiếu authority/evidence hoặc divergence chưa được phép xử lý → **BLOCKED** đối với reconciliation/publication. |

**Candidate → commit → publish.** Bind verification với **exact full candidate** theo mục 5; kiểm accepted scope, actual changed paths, relevant dirty content và material inputs. Stage **explicit authorized paths** (không `git add .` / `git add -A`), kiểm **toàn bộ** staged diff/index để không mang theo unrelated pre-staged changes; commit chỉ khi được phép và candidate/checks đạt. Kiểm resulting commit identity/content; trước push fresh-query remote, dùng native Git ancestry/fast-forward và inspect **toàn bộ remote..HEAD commit range** cùng authorization, kể cả commits có sẵn từ phiên trước. Chỉ non-force push đúng origin/branch khi an toàn. Server-side non-fast-forward rejection là guard, không thay fresh precheck; remote đổi/reject → inspect current state trước quyết định tiếp. Fresh-query remote **sau push**: chỉ xác nhận `PUBLISHED` nếu chứng minh expected commit có trong published history (fetch/kiểm ancestry khi tip đã đổi), không dựa duy nhất vào push stdout/timeout. Candidate hoặc inputs material đổi → reverify phần bị ảnh hưởng.

**Completion evidence** — task acceptance, publication, HEAD synchronization và global cleanliness là những khẳng định **khác nhau**:

| Trạng thái hiện có | Điều kiện |
| --- | --- |
| `VERIFIED` | Accepted task outcome đã được kiểm trên exact candidate; chưa chứng minh commit/publish. |
| `COMMITTED_LOCAL` | Authorized commit tồn tại local; chưa xác nhận GitHub. |
| `PUBLISHED` | Fresh remote evidence chứng minh expected commit trong published history; không suy ra HEAD equality hoặc clean. |
| `SYNCED_CLEAN` | `PUBLISHED`, actual local HEAD **bằng fresh remote HEAD**, `git status --porcelain=v1 -uall` **không có output**, không unresolved Git operation; ignored local-only config được bảo toàn. |
| `PARTIAL` | Một phần task outcome hoàn thành nhưng acceptance còn thiếu; báo phần thiếu. |
| `BLOCKED` | Thiếu authority/capability/evidence thiết yếu hoặc không thể tiếp tục an toàn. |

Unrelated dirty state có thể cùng tồn tại với `VERIFIED`, `COMMITTED_LOCAL` hay `PUBLISHED`; **không** tự dọn để đạt `SYNCED_CLEAN`. Task DONE không tự chứng minh publication, deployment hoặc release authorization. Không tạo daemon, watcher, hook, cron, background sync hay dependency mới; Markdown không tự enforce Git.

**Unknown effects / exceptions.** Khi consequential operation có thể đã thành công nhưng mất acknowledgement: giữ original operation identity/locator (nếu có) và expected postcondition, inspect thực tế từ local/remote và kết quả operation gốc nếu recoverable; retry **chỉ** khi chứng minh effect chưa xảy ra và preconditions vẫn đúng, hoặc operation contract chứng minh replay-safe. Chưa xác định được → report outcome unknown, **BLOCKED** cho thao tác phụ thuộc; không blind retry. Không áp dụng transaction-recovery ceremony lên routine local edits có kết quả kiểm được trực tiếp. Remote unavailable không đồng nghĩa push thất bại hoặc publication thành công.

**Cross-session continuation.** Handoff gốc + actual target/ref/HEAD, authorized scope, changed paths/dirty state, candidate-bound checks và kết quả, unpublished commit range khi liên quan, unresolved effect/locator và next authorized action; phiên sau tự rebind state/authority trước khi tiếp tục. Escalate khi identity/permission sai, conflicting requirements, material impact chưa rõ, security/data risk, irrecoverable migration, mandatory independent review hoặc verification chưa đủ; không tự cấp quyền.

## 4. Engineering Skills & ECC — HOW có điều kiện, ưu tiên target-native

**Thứ tự ưu tiên về authority/applicability, không phải pipeline bắt buộc:** target requirements/restrictions → target-native conventions/code/tooling → [Engineering Skills](https://github.com/phatnguyen03022001/engineering-skills) cho engineering HOW nếu phù hợp/khả dụng → ECC upstream/reference tùy chọn → target-native verification. Target sở hữu product/implementation truth; [SDLC.md](SDLC.md) sở hữu lifecycle/gates; tài liệu này sở hữu execution authority/delegation/session governance. Skills, installed plugins, ECC và tool availability **không cấp quyền hoặc chứng minh activation**. Không bắt cài plugin hoặc áp dụng pattern HOW để SDLC hoạt động.

**Kích hoạt theo nhu cầu thực tế:**

- **Trivial / routine:** typo, formatting, config nhỏ với contract rõ → làm theo native workflow và focused checks; **không bắt ECC discovery**.
- **Ordinary:** bug cô lập, cải tiến nhỏ, feature có accepted requirements → native workflow, root cause và affected regression; chỉ đọc Engineering Skills/ECC nếu cải thiện rõ reasoning, implementation hoặc verification.
- **Material / specialized:** architecture, security boundary, migration, cross-module contract, complex reliability/refactor → **chọn Engineering Skills phù hợp hoặc ECC reference khi có và áp dụng được**; không nạp toàn catalog. Task nhỏ vẫn có thể dùng ECC khi có capability gap/rủi ro thật.

Engineering Skills và ECC tham khảo có thể hữu ích cho clarification, change-impact, module/contract design, security, debugging, testing, verification hoặc documentation integrity; **không bắt mọi task chọn skill**. Bắt đầu 1–2 skills là *heuristic* khi đã quyết định dùng ECC, **không phải quota/hard cap**; nạp thêm khi risk/capability đòi hỏi.

**Chỉ khi thực sự dùng ECC:** xác minh skill tồn tại, `name/source@revision` hoặc content identity phù hợp (checkout có thể khác Foundation pin; dirty thì SHA không đủ); đọc **phần áp dụng**, ghi provenance/adopted hoặc material skipped guidance khi cần truy xuất; bỏ qua Claude-only hooks/orchestration/config, schema đầu ra/coverage threshold không thuộc target policy và ví dụ không tương thích; chuyển HOW sang commands/conventions của target và kiểm kết quả thực. Không coi skill hay metadata là proof of PASS.

Nếu ECC không truy cập được mà không có **yêu cầu cụ thể bắt buộc** sử dụng capability đó, tiếp tục với engineering practices và target-native checks, **không khai đã dùng ECC**, không clone/cài đặt chỉ để qua quy trình. Nếu task bắt buộc capability không thể truy cập hoặc thay thế phù hợp, báo blocker thật. Không tự kích hoạt skill cho mọi task, đếm số skill làm KPI, nạp full catalog, cài Claude hooks/agents, sinh file, thêm dependency/infra/automation chỉ vì skill đề xuất; mọi thay đổi vẫn cần scope, authority và admission theo [REPOSITORY.md](REPOSITORY.md).

## 5. Verification, review, report

- **READY:** đúng target/authority, outcome/contract/acceptance, dependency và test surface rõ.
- **VERIFY:** chạy checks liên quan trên môi trường thật; private helper → unit; shared module → direct/transitive consumers; API/schema → contract/integration & compatibility; auth/payment/migration/config → security, data, failure-path và broader regression theo rủi ro. Dynamic routing/unknown consumers → mở rộng checks hoặc báo không đủ bằng chứng. Không bắt buộc full E2E sau mọi sửa.
- **DONE:** review diff/scope, acceptance đủ evidence, không còn blocker nghiêm trọng; **tests green ≠ đúng nghiệp vụ**.
- **RELEASE:** gate riêng của [SDLC.md](SDLC.md), không suy ra từ DONE.

**Candidate/evidence binding:** Candidate clean, đã commit chỉ dùng exact commit SHA sau khi kiểm working content khớp; candidate dirty cần base HEAD + affected paths + staged/unstaged state + relevant untracked content + diff/content identity có thể đối chiếu. Gắn mỗi verification với candidate thực kiểm, command/verifier profile, observed result, material environment/toolchain, inputs và assumptions. Đối chiếu candidate **sau** khi kiểm; content, test conditions hoặc material inputs thay đổi thì chỉ tái dùng evidence khi chứng minh còn áp dụng, nếu không phải reverify phần bị ảnh hưởng. Không có candidate identity đáng tin → **NOT VERIFIED**; không cần fingerprint store hay script mới.

**Independent review chỉ khi bắt buộc:** `Reviewer / Candidate / Evidence / Decision`. Reviewer phải độc lập thật nếu policy đòi hỏi; nhãn hoặc metadata không chứng minh independence. Đánh giá đúng candidate và evidence có thể kiểm; thiếu mandatory independent evidence → gate **không PASS**. Self-review chỉ là internal check.

**Báo kết quả theo rủi ro trong chat hoặc tracker có sẵn**, bằng các nhóm thông tin logic (không bắt buộc format):

- **Result:** `PASS | FAIL | NOT VERIFIED`, tương ứng acceptance đã thực sự chứng minh.
- **Changed:** phần thay đổi và affected consumers khi có.
- **Evidence:** candidate đã kiểm, checks/profile, kết quả quan sát được và điều kiện kiểm chứng liên quan.
- **Remaining issues:** checks chưa chạy/lý do, unknowns, risks và material blockers khi có.
- **Git publication state:** tình trạng commit/push/sync thực tế khi liên quan; dùng đúng các trạng thái đã định nghĩa tại [Git synchronization](#git-synchronization--canonical-execution-contract).

Với task đọc hoặc thay đổi nhỏ không publication, bỏ các nhóm không liên quan. Với công việc consequential, **phải giữ exact candidate identity**, evidence gắn với candidate/environment/inputs, skipped checks và material blockers; bổ sung reviewer độc lập, release/rollback hoặc unknown-effect status khi liên quan. `PASS` đòi hỏi acceptance evidence thật; `PUBLISHED` không suy ra `SYNCED_CLEAN`, và `DONE` không suy ra deployment.

**Adversarial check gọn trước khi kết luận:** (a) AC nào vẫn sai dù tests xanh? (b) tác động tới consumer/config/permission gián tiếp nào chưa kiểm? (c) state/remote/test output nào có thể stale? (d) failure, rollback, data leakage hoặc unauthorized change nào có thể xảy ra? (e) thao tác nào vẫn UNKNOWN? Chỉ mở rộng kiểm tra khi có failure mode cụ thể; không giả vờ đã có independent reviewer.

**PASS chỉ khi bằng chứng đáp ứng toàn bộ acceptance liên quan.** Independent review phải được gắn đúng nguồn; release permission tách riêng. Không sinh report Markdown, task YAML hay lịch sử chat vào repo chỉ để trông “đủ quy trình”.
