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
Verification: [checks, evidence links, N/A có lý do]
Risk / rollback: [khi có tác động dữ liệu/production]
Result: PASS | FAIL | NOT VERIFIED + evidence + blockers
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

## 3. Routing tham chiếu (tài liệu OpenAI kiểm chứng ngày 2026-10-31)

| Tình huống | ChatGPT (nếu hiện trong picker) | Codex/Work (nếu khả dụng) | Start effort |
| --- | --- | --- | --- |
| XS, chỉnh docs/reformat/summarize | GPT-6 Instant | GPT-6 Luna; Terra cho sửa code/visual nhỏ | Low |
| S–M, spec rõ/bug fix/feature có test | GPT-6 Sol Medium | GPT-6.1 Sol Medium hoặc Terra khi task hẹp | Medium |
| M–L, multi-module, contract/invariants | GPT-6 Sol High | GPT-6.1 Sol High hoặc GPT-6 Astra Medium | Medium → High nếu cần |
| L, root cause mơ hồ, security/data/architecture risk | GPT-6 Sol High/Extra High hoặc Pro (nếu có) | GPT-6 Astra Medium/High | Escalate dựa trên evidence |
| XL | Phân rã trước | Phân rã trước, route từng task | Không chạy nguyên khối |

- Đây là **guidance, không phải hardcoded mapping**; tính khả dụng phụ thuộc plan/workspace/version, không đoán từ tên GPT. Chat picker Instant/Medium/High/Extra High/Pro khác API/Codex `reasoning.effort` low/medium/high/xhigh/max và **mỗi model có tập mức riêng**.
- Chọn cấu hình nhẹ nhất **qua được acceptance + regression eval**. Model mạnh hơn không bù được thiếu context, quyền, tools, environment hoặc test data. Khi thất bại: sửa thiếu input/scope trước, rồi mới escalate.
- Chỉ dùng parallel agents, independent review, high/max effort khi rủi ro/chất lượng thực sự cần. Đo tổng **chi phí hoàn thành có bằng chứng** (latency, retry, review, model allowance), không tối ưu đơn giá token một cách mù quáng.
- Không chép tên model và mức effort vào `AGENTS.md` target vì dễ stale; kiểm lại tại thời điểm thực thi.

## 4. Gates cho task

1. **READY:** scope, baseline, owner/authority, acceptance và dependencies đủ rõ; khi thiếu làm investigation có giới hạn.
2. **VERIFY:** test đúng failure mode, regression, security/compatibility khi liên quan. Không có tool/evidence = **NOT VERIFIED**.
3. **DONE:** evidence gắn với thay đổi, diff trong scope, không còn blocker nghiêm trọng. Không đồng nhất “tests green” với sản phẩm đúng nghiệp vụ.
4. **RELEASE:** production chỉ qua pre-release UAT/sign-off, migration, smoke/monitor/rollback như [SDLC.md](SDLC.md); DONE không tự động là DEPLOYED.

## 5. Nguồn chính thức (kiểm lại trước khi cố định routing)

- [ChatGPT GPT-6 picker](https://help.openai.com/en/articles/20001354-gpt-6-and-other-models-in-chatgpt)
- [OpenAI model selection](https://developers.openai.com/api/docs/guides/model-selection)
- [Reasoning effort](https://developers.openai.com/api/docs/guides/reasoning)
- [Work/Codex model & usage](https://help.openai.com/en/articles/20001516-managing-usage-with-gpt-6-astra-in-work-and-codex)
- [Avoiding bloated AGENTS.md/prompts](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra)
