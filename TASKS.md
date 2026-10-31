# Task Contract — Scope, Effort & Model Routing

> Quy định **WHAT / DONE / EVIDENCE** cho ChatGPT, Codex và task executor; ECC quyết định **HOW**. Không tạo prompt framework, router mới hay agent orchestration.

## 1. Task đủ rõ để giao

Task là **một outcome có thể xác minh**; không phải một phase khổng lồ hay một danh sách file phải sửa.

```md
Outcome: [hành vi/kết quả cần đạt]
Target & baseline: [repo, branch/commit, môi trường xác nhận được]
Scope & non-goals: [giới hạn; cái gì không đổi]
Acceptance: [tiêu chí test/đo được]
Constraints: [data/privacy/security/compatibility/permission]
Verification: [affected tests/gates, evidence links; N/A có lý do]
Risk / rollback: [khi có tác động dữ liệu/production]
Result: PASS | FAIL | NOT VERIFIED + changed surface / consumers / executed checks / residual risk
```

Có thể **link sang issue/spec** thay cho lặp thông tin. Không ép viết đủ trường cho thay đổi cơ học. Ưu tiên vertical slice hoàn chỉnh (FE+BE+DB+test cần thiết cho cùng một hành vi) thay vì chia cơ học theo layer. Nếu chưa biết root cause/contract thì làm investigation **read-only** trước, không giả sử đã implement.

## 2. Sizing của task ≠ reasoning effort

Đánh giá: ambiguity, boundaries/invariants, dependencies, reversibility, blast radius, rủi ro data/security, khả năng test. Không ước lượng bằng LOC, số file hay thời gian agent “chắc chắn” hoàn thành.

| Task size | Dạng công việc | Hành động mặc định |
| --- | --- | --- |
| **XS** | Chỉnh hẹp, cơ học, test sẵn | Làm trực tiếp, verify |
| **S** | Bug fix cô lập / một invariant rõ | Một task có regression |
| **M** | Một feature/flow xuyên vài lớp, contract đã rõ | Một vertical task có acceptance |
| **L** | Nhiều boundary, migration, khó rollback hoặc quyết định chưa rõ | Investigation + quyết định + chia khi cần |
| **XL** | Nhiều feature, rủi ro/nhóm phụ thuộc | **Chia milestones/task**, không đẩy nguyên khối |

Khối lượng triển khai/review/rollout (**task effort**) khác tài nguyên suy luận của model (**reasoning effort**). Nếu phải forecast người/ngày hoặc ngân sách, dùng analog tasks, dải giá trị + độ tin cậy, và cập nhật dự báo khi có bằng chứng. Không áp “15 phút mỗi task”, một mức token cố định hoặc quota giả.

## 3. Operating model ưu tiên: ChatGPT Architect → Luna Executor

**Preference mặc định của owner**, không phải quyền thực thi hoặc điều kiện mọi task đều phải dùng hai model:

1. **ChatGPT (Architect):** đọc target canonical facts, xác định root cause / boundary, challenge scope, chọn smallest correct change; viết task với acceptance, risk, dependencies, evidence, và điểm dừng. **Không đồng thời làm “global memory” của mọi feature**; nạp đúng context cần thiết.
2. **GPT-6 Luna (Executor):** nhận task nhỏ có contract rõ; đọc đúng files/code cần sửa, sử dụng ECC (nếu đã cài) cho **HOW**, implement + regression + report evidence. Không tự đổi scope, tự chọn infra hoặc ghi PASS khi chưa chạy checks.
3. **Architect review theo rủi ro:** kiểm các invariant, data/security, compat, diff và evidence. Không review từng dòng boilerplate mà lint/tests đã chứng minh.
4. **Escalation:** nếu Luna fail do thiếu context → sửa input; do task quá rộng → tách; nếu vẫn fail có chứng cứ / yêu cầu reasoning đa boundary → giao Sol/Astra **cho đúng phần khó** (hoặc để ChatGPT Architect ra quyết định), không tự nâng model toàn bộ project.
5. Không ép model tùy task: XS có thể tự xử lý ngay, S–M ưu tiên Luna; L/XL phân tích/chia trước. **Cost = tokens + retry + review + elapsed time**, không chỉ số token của executor.

**Task handoff ngắn gọn:** `Outcome | Target/ref | Scope/non-goals | Acceptance | Test/evidence | Risk/rollback`. Mọi trường đã rõ trong canonical spec/issue có thể link thay vì nhắc lại. Giao việc bằng một outcome, không giao “fix all / cleanup all / 11 phases”.

## 4. Routing tham chiếu (tài liệu OpenAI kiểm chứng ngày 2026-10-31)

| Tình huống | ChatGPT (nếu hiện trong picker) | Codex/Work (nếu khả dụng) | Start effort |
| --- | --- | --- | --- |
| XS, đơn giản / mechanical | Architect tùy cần | **Luna** executor; có thể làm trực tiếp | Low |
| S–M, scoped bug/vertical feature có test | **ChatGPT architect khi cần** | **Luna** executor mặc định | Low → Medium |
| M–L, multi-module/compatibility | ChatGPT architect scope & risk | Luna cho slices đã xác định; Sol nếu có blocker thực | Medium, đo trên test |
| L, incident/security/data architecture chưa rõ | ChatGPT reasoning High khi có | Sol/Astra cho **investigation hẹp** nếu Luna không phù hợp | High khi được chứng minh |
| XL | Architect phân rã milestones | Luna cho từng task đã được scope; escalate phần khó | Không giao nguyên khối |

- Đây là **guidance, không phải hardcoded mapping**; tính khả dụng phụ thuộc plan/workspace/version, không đoán từ tên GPT. Chat picker Instant/Medium/High/Extra High/Pro khác API/Codex `reasoning.effort` low/medium/high/xhigh/max và **mỗi model có tập mức riêng**.
- Chọn cấu hình nhẹ nhất **qua được acceptance + regression eval**. Model mạnh hơn không bù được thiếu context, quyền, tools, environment hoặc test data. Khi thất bại: sửa thiếu input/scope trước, rồi mới escalate.
- Chỉ dùng parallel agents, independent review, high/max effort khi rủi ro/chất lượng thực sự cần. Đo tổng **chi phí hoàn thành có bằng chứng** (latency, retry, review, model allowance), không tối ưu đơn giá token một cách mù quáng.
- **GPT-6 Luna là lựa chọn executor ưu tiên**, không phải mức quyền/hiệu năng được bảo đảm. Không chép tên model và mức effort vào `AGENTS.md` target vì dễ stale; kiểm lại tại thời điểm thực thi.

## 5. Gates cho task

1. **READY:** scope, baseline, owner/authority, acceptance và dependencies đủ rõ; khi thiếu làm investigation có giới hạn.
2. **VERIFY:** test đúng failure mode, regression, security/compatibility khi liên quan. Với shared/contract change, kiểm affected consumers theo [SDLC.md](SDLC.md); khi mapping không đủ, chạy broader checks hoặc báo NOT VERIFIED. Không có tool/evidence = **NOT VERIFIED**.
3. **DONE:** evidence gắn với thay đổi, diff trong scope, không còn blocker nghiêm trọng. Không đồng nhất “tests green” với sản phẩm đúng nghiệp vụ.
4. **RELEASE:** production chỉ qua pre-release UAT/sign-off, migration, smoke/monitor/rollback như [SDLC.md](SDLC.md); DONE không tự động là DEPLOYED.

**Change-impact evidence tối thiểu khi báo DONE:** `changed surface → affected consumers (kể cả transitive/indirect) → checks executed + result/link → skipped checks + rationale → residual risk`. Với thay đổi đơn giản, một dòng đủ; không sinh report Markdown riêng. Task chọn mức kiểm chứng theo blast radius, không lấy số file sửa làm chuẩn. **NOT VERIFIED không được diễn giải thành PASS**; test xanh nhưng scope sai/thiếu consumer vẫn chưa đủ evidence.

## 6. Nguồn chính thức (kiểm lại trước khi cố định routing)

- [ChatGPT GPT-6 picker](https://help.openai.com/en/articles/20001354-gpt-6-and-other-models-in-chatgpt)
- [OpenAI model selection](https://developers.openai.com/api/docs/guides/model-selection)
- [Reasoning effort](https://developers.openai.com/api/docs/guides/reasoning)
- [Work/Codex model & usage](https://help.openai.com/en/articles/20001516-managing-usage-with-gpt-6-astra-in-work-and-codex)
- [Avoiding bloated AGENTS.md/prompts](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra)
