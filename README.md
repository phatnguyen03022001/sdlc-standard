# SDLC Standard — Agent-ready Mini

Chuẩn Markdown gọn cho phát triển sản phẩm với **ChatGPT Architect → ChatGPT hoặc Codex Executor**. Kết hợp vòng đời SDLC, quyền thực thi an toàn và ECC chuyên trách **HOW**; không phải agent runtime, bộ cài, chứng nhận ISO hay cơ chế tự động cấp quyền.

| Nguồn duy nhất | Sở hữu |
| --- | --- |
| [SDLC.md](SDLC.md) | 11 nhóm lifecycle, 8 trục chất lượng, quality/release gates |
| [TASKS.md](TASKS.md) | ChatGPT Architect → ChatGPT/Codex, capability routing, material design gate, handoff, authority, continuity, ECC HOW |
| [REPOSITORY.md](REPOSITORY.md) | Ownership, tài liệu tối thiểu, chống rác/drift |
| [research/README.md](research/README.md) | Nghiên cứu tham khảo, không tự thành quyết định áp dụng |

## Cách dùng (không cần cài gì)

1. **Bind target:** xác minh đúng repo/branch/HEAD, instructions và trạng thái local; target sở hữu sự thật sản phẩm.
2. **Architect:** phân loại intent (READ_ONLY/DESIGN/IMPLEMENT/REVIEW/SECURITY/RELEASE), chọn outcome/gate trong [SDLC.md](SDLC.md), chốt material decisions khi cần, root cause, trade-off, phạm vi nhỏ nhất và acceptance; giao task theo [TASKS.md](TASKS.md).
3. **Executor:** dùng **ChatGPT nếu phiên có công cụ truy cập/thực thi**, hoặc **Codex** khi cần làm trực tiếp trên workspace. Chỉ nạp context và [ECC](https://github.com/affaan-m/everything-claude-code) skill phù hợp, không cài/copy toàn bộ ECC.
4. **Verify:** tests theo blast radius, kiểm diff và bằng chứng. Phân biệt kiểm tra bản thân với independent review.
5. **Publish khi được phép riêng:** chỉnh sửa không tự cấp quyền commit/push/deploy. Không có tool/evidence thì báo **NOT VERIFIED**, không giả PASS.

**Mặc định:** một task nhỏ, một target, một writer; không bắt buộc hai phiên chat, YAML, scripts, hooks, CI mới, task/report files hoặc nhiều agent. Dùng source of truth sẵn có; Git/issue/CI giữ lịch sử. Foundation bên ngoài là **tùy chọn**, không phải dependency.

Bản mini v2.1 · 2026-11-02.
