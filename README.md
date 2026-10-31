# SDLC Standard

Chuẩn **WHAT / WHEN / DONE** tối giản cho web app phức tạp: 11 giai đoạn, 8 trục xuyên suốt và quality gates có bằng chứng. Không phải chứng nhận ISO, framework thực thi hoặc generator.

| Tài liệu chính thức | Chủ sở hữu nội dung |
| --- | --- |
| [SDLC.md](SDLC.md) | Phase, checklist, deliverables, quality gates, 8 trục |
| [REPOSITORY.md](REPOSITORY.md) | Ownership, docs structure, naming, chống trùng lặp với ECC |
| [TASKS.md](TASKS.md) | Định nghĩa task, sizing, effort, ChatGPT/Codex routing, verification |

**Chỉ 4 Markdown tại root** (kể cả README); không thêm agent/skills/rules/commands/scripts/CI. Git là lịch sử; không lưu bản sao report/state theo từng turn.

## Sử dụng với target repo

1. Xác định mục tiêu và gate theo [SDLC.md](SDLC.md), không ép waterfall.
2. **Kiểm kê tài liệu/tracker có sẵn**, tái sử dụng owner theo [REPOSITORY.md](REPOSITORY.md); không scaffold mặc định.
3. Giao một outcome có acceptance/evidence theo [TASKS.md](TASKS.md), chọn model & reasoning effort phù hợp.
4. **ECC = HOW** (coding, testing, review, debugging, workflow). Không fork ECC vào đây. **Agent Foundation nếu có = authority/router/execution boundary**, không phải bắt buộc.
5. Report PASS/FAIL/NOT VERIFIED theo bằng chứng, không tự chấm chất lượng 10/10.

Target repo sở hữu **business facts, code, contracts, tests, release decisions**. Chuẩn này chỉ đặt kỳ vọng về kết quả/gates; không có quyền ghi code hay phê duyệt release. Giữ một nguồn sự thật mỗi chủ đề.

**v1.1 · 2026-10-31**
