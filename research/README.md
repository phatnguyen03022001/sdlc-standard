# Research Index — Infrastructure, Third-party & Technology

> **Vai trò:** nguồn tri thức **tham khảo**, giúp ChatGPT Architect ra quyết định cho **feature cụ thể**. Không tự biến research thành architecture hiện hành của target; ECC chịu trách nhiệm HOW.

| Chủ đề | Câu hỏi chính | Tài liệu |
| --- | --- | --- |
| Infrastructure | Chọn deployment, DB, caching, queue, HA, observability và cách scale **theo ngưỡng** nào? | [infrastructure.md](infrastructure.md) |
| Third-party | Khi nào mua/dùng dịch vụ ngoài, rủi ro vendor, cost, security và exit? | [third-party.md](third-party.md) |
| Feature-based & tech selection | Tại sao chọn TypeScript/Next.js, Go, Rust, Python… cho feature thay vì theo trend? | [technology-selection.md](technology-selection.md) |

**Technology Catalog (tham khảo bên ngoài):** [phatnguyen03022001/technology-catalog](https://github.com/phatnguyen03022001/technology-catalog) — tra cứu languages, libraries, frameworks, providers và các so sánh theo workload. Chỉ dùng để lập shortlist; đối chiếu nguồn hiện hành và requirements của target trước khi quyết định, không coi catalog là architecture authority.

**Format chuẩn mỗi nghiên cứu:** `Question → Baseline / recommendation → Alternatives & trade-offs → Decision triggers → Verification plan → Primary sources`.

**Metadata tối thiểu:** `Status: reference / decision-required / superseded`; `Verified: YYYY-MM-DD`; `Scope / assumptions`; `Recheck triggers`. Định nghĩa:
- **reference:** đã đối chiếu tài liệu chính thức, nhưng **chưa phải quyết định áp dụng**.
- **decision-required:** thiếu dữ liệu workload/thực tế để lựa chọn; không bịa thông số.
- **superseded:** có bằng chứng mới hoặc canonical ADR thay thế; xóa/gộp sau khi chuyển link cần thiết, không tạo nghĩa trang docs.

## Admission & refresh rules

- Mỗi tài liệu giải quyết **một nhóm câu hỏi bền vững**, không lập file theo provider, framework version, bài blog, prompt hay ngày.
- **Source first:** link ưu tiên upstream/official, phân biệt spec fact với judgment. Giá, quota, retention, region, SLA, policy, auth SDK: **phải xác minh lại tại thời điểm quyết định**.
- Chỉ ghi những gì phục vụ trade-off, failure mode hoặc acceptance, không chép nguyên docs/vendor marketing. Không điền bảng benchmark tưởng tượng.
- Mọi đánh giá phải có **điều kiện thay đổi lựa chọn** và **cách đo/verify**; “X tốt nhất” không đủ.
- Khi feature chọn xong: kết luận trở thành **target-owned ADR/feature spec** (nếu cần); research giữ ma trận tổng quát, không chứa trạng thái dự án, secrets hay account IDs.
- Nếu công nghệ/nhà cung cấp thay đổi, audit đúng mục bị ảnh hưởng; review khi có breaking changes, incident hoặc trước quyết định mới. Không đặt cron/maintenance churn khi chưa có consumer.

**Owner (logical):** Architect/maintainer của repository đang dùng kiến thức. Không tạo reviewer account, YAML database, agent workflow hoặc hệ thống sync mới.

**Reference baseline:** 2026-10-31. Không áp dụng trực tiếp các con số vận hành chưa có đo đạc.
