# SDLC Standard — Agent-ready Mini

Chuẩn Markdown gọn cho phát triển sản phẩm với **ChatGPT Architect → Chat/Work/Codex Executor**. Kết hợp vòng đời SDLC, quyền thực thi an toàn và [Engineering Skills](https://github.com/phatnguyen03022001/engineering-skills) chuyên trách **HOW** (ECC là upstream/reference tùy chọn); không phải agent runtime, bộ cài, chứng nhận ISO hay cơ chế tự động cấp quyền.

| Nguồn duy nhất | Sở hữu |
| --- | --- |
| [SDLC.md](SDLC.md) | 11 nhóm lifecycle, 8 trục chất lượng, quality/release gates |
| [TASKS.md](TASKS.md) | Architect → Chat/Work/Codex, environment/mode routing, conditional Agent Team, session handoff, authority, Git/evidence, Skills/ECC tham khảo |
| [REPOSITORY.md](REPOSITORY.md) | Ownership, tài liệu tối thiểu, chống rác/drift |
| [research/README.md](research/README.md) | Nghiên cứu tham khảo, không tự thành quyết định áp dụng |

## Cách dùng (không cần cài gì)

1. **Bind target:** xác minh đúng repo/branch/HEAD, instructions và trạng thái local; target sở hữu sự thật sản phẩm.
2. **Architect:** phân loại intent (READ_ONLY/DESIGN/IMPLEMENT/REVIEW/SECURITY/RELEASE), chọn outcome/gate trong [SDLC.md](SDLC.md), chốt material decisions khi cần, root cause, trade-off, phạm vi nhỏ nhất và acceptance; giao task theo [TASKS.md](TASKS.md).
3. **Executor:** chọn Chat/Work/Codex theo workload, tool/quyền thực có; Codex ưu tiên cho native repo work. Single Agent mặc định; Agent Team chỉ khi qua admission trong [TASKS.md](TASKS.md). Ưu tiên target-native workflow, dùng Engineering Skills khi hữu ích; [ECC](https://github.com/affaan-m/everything-claude-code) chỉ là reference, không bắt cài plugin.
4. **Verify:** tests theo blast radius, kiểm diff và bằng chứng. Phân biệt kiểm tra bản thân với independent review.
5. **Publish khi được phép riêng:** chỉnh sửa không tự cấp quyền commit/push/deploy. Không có tool/evidence thì báo **NOT VERIFIED**, không giả PASS.

**Mặc định:** một task nhỏ, một target, một writer; không bắt buộc hai phiên chat, YAML, scripts, hooks, CI mới, task/report files hoặc nhiều agent. Dùng source of truth sẵn có; Git/issue/CI giữ lịch sử. Foundation bên ngoài là **tùy chọn**, không phải dependency.

**Status: STABLE / FROZEN BASELINE.** Đóng chương trình simplification; chỉ thay đổi khi có lỗi correctness/safety P0/P1 đã chứng minh, thay đổi material của platform/tool áp dụng, contract/reference thiết yếu bị hỏng, yêu cầu security/compatibility material hoặc tiến hóa kiến trúc có lý do. Mọi thay đổi cần authorization rõ ràng của owner theo [TASKS.md](TASKS.md); freeze là quyết định bảo trì, không cấm sửa lỗi cần thiết và không mở lại vì style, suy đoán hay giảm Markdown.

Bản mini v2.1 · 2026-11-02.
