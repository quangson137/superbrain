# 9-Layer Checklist — Tiêu Chí Đánh Giá Skill

## Layer 0 — Use Case & Trigger Map
- [ ] Skill giải quyết công việc cụ thể, không quá rộng
- [ ] Xác định rõ người dùng chính
- [ ] Có trigger dương (khi nào kích hoạt)
- [ ] Có trigger âm (khi nào KHÔNG kích hoạt)

## Layer 1 — Metadata
- [ ] Có YAML frontmatter với `name` và `description`
- [ ] `description` cụ thể, bám ngôn ngữ người dùng thật
- [ ] Có trigger phrases (cả tiếng Việt lẫn Anh nếu cần)
- [ ] Có trigger âm (Do NOT use for...)
- [ ] Có `version`
- [ ] Tổng description 50–100 từ

## Layer 2 — Core SKILL.md
- [ ] Phần 1: PURPOSE — 2–3 câu
- [ ] Phần 2: WHEN TO USE — trigger dương + âm
- [ ] Phần 3: EXPECTED INPUTS — bắt buộc/tuỳ chọn
- [ ] Phần 4: WORKFLOW — từng bước, chỉ resource
- [ ] Phần 5: OUTPUT FORMAT — cấu trúc, mẫu
- [ ] Phần 6: RESOURCE USAGE — khi nào đọc gì
- [ ] Phần 7: GUARDRAILS — giới hạn
- [ ] Phần 8: FINAL CHECK — checklist tự kiểm
- [ ] Dưới 500 dòng / ~3.000 từ
- [ ] Không chứa tri thức dài nên tách sang references

## Layer 3 — References
- [ ] Có thư mục references/ nếu cần tri thức nền
- [ ] SKILL.md ghi rõ khi nào đọc file nào
- [ ] Không trùng nội dung với SKILL.md

## Layer 4 — Examples
- [ ] ≥1 good example (input → output)
- [ ] ≥1 anti-example (mẫu xấu + phân tích lỗi)
- [ ] Khuyến nghị: annotated example

## Layer 5 — Scripts & Tools
- [ ] Nếu có logic lặp → có script
- [ ] Script có mục đích rõ ràng

## Layer 6 — Assets & Templates
- [ ] Template tách riêng khỏi references
- [ ] Nếu skill tạo file → có mẫu trong assets/

## Layer 7 — Output Contract & QC
- [ ] Có quy định cấu trúc đầu ra
- [ ] Có tiêu chí ĐẠT / CẦN SỬA / KHÔNG ĐẠT
- [ ] Có self-check checklist 5–7 mục

## Layer 8 — Governance
- [ ] Có version number
- [ ] Khuyến nghị: owner, changelog
- [ ] Cho cá nhân: chỉ cần version là đủ
