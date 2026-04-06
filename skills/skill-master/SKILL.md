---
name: skill-master
description: >
  Use this skill when the user asks to 'review a skill', 'check skill format',
  'đánh giá skill', 'kiểm tra skill', 'audit skill', 'chấm điểm skill',
  'fix my skill', 'sửa skill', 'chuẩn hoá skill', 'adapt skill to standard',
  'refactor skill', 'tạo skill mới', 'create a skill', 'viết skill',
  'help me write a skill', 'skill này đã chuẩn chưa?', 'what's missing?',
  'make this skill production-ready', or wants to build/review/fix any skill
  following the 9-layer Skill Engineering framework.
  Do NOT use for: general prompt review, reviewing code unrelated to skill structure.
version: 1.1.0
---

# SKILL: Skill Master — Create, Review & Fix Skills Theo Chuẩn 9 Layer

## 1. PURPOSE
Skill này có 3 chế độ:
- **Create**: Hỏi intent → Phỏng vấn → Viết SKILL.md chuẩn 9 layer → Sinh examples → Tự review.
- **Review**: Đánh giá skill theo framework 9 Layer, chấm điểm 6 trục, chỉ ra lỗi.
- **Fix**: Dựa trên kết quả review, tự động viết lại / bổ sung các phần thiếu
  để đưa skill về đúng format chuẩn.

Người dùng có thể chọn bất kỳ chế độ nào, hoặc kết hợp (create → review, review → fix).

## 2. WHEN TO USE
✅ Dùng khi:
- **Create**: Muốn tạo skill mới từ đầu, "viết skill cho tôi", "tôi muốn đóng gói workflow này thành skill"
- **Review**: Kiểm tra / audit / chấm điểm skill đã có
- **Fix**: Sửa / chuẩn hoá / refactor skill cũ cho đúng format
- Hỏi "skill này đã chuẩn chưa?" rồi muốn sửa luôn
- Muốn nâng cấp skill từ mức Chưa đạt / Cơ bản lên Tốt / Trưởng thành

❌ KHÔNG dùng:
- Review prompt đơn lẻ không phải skill
- Review code không liên quan đến cấu trúc skill

## 3. EXPECTED INPUTS
**Chế độ Create:**
- Mô tả ý tưởng skill (bắt buộc) — có thể ngắn gọn, sẽ phỏng vấn thêm
- Loại skill: technique / pattern / reference (tuỳ chọn)

**Chế độ Review / Fix:**
- Đường dẫn tới thư mục skill HOẶC nội dung SKILL.md (bắt buộc)
- Chế độ: `review` | `fix` | `review+fix` (tuỳ chọn, mặc định: review+fix)
- Loại skill: cá nhân / team / doanh nghiệp (tuỳ chọn, mặc định: cá nhân)

## 4. WORKFLOW

### PHASE 0 — CREATE (chỉ chạy khi chế độ là `create`)

**Bước 1: Capture Intent**
- Nếu conversation đã có workflow cụ thể → rút trích trước, sau đó hỏi xác nhận
- Hỏi 4 câu hỏi cốt lõi (chỉ hỏi những gì chưa rõ):
  1. Skill giải quyết việc gì? Cho ai?
  2. Trigger khi nào / KHÔNG trigger khi nào?
  3. Output mong muốn trông như thế nào?
  4. Loại skill: **technique** (có bước rõ) / **pattern** (mental model) / **reference** (tài liệu tra cứu)?

**Bước 2: Interview & Research**
- Hỏi về edge cases, inputs/outputs bắt buộc vs tuỳ chọn
- Nếu có MCP/tool liên quan → nghiên cứu trước khi viết
- Xác nhận với user trước khi sang bước 3

**Bước 3: Viết SKILL.md theo 9 Layer**

*Layer 1 — Metadata:*
- Đọc `references/metadata-formula.md` để viết description đúng công thức
- Description bắt đầu bằng `Use this skill when...`, liệt kê trigger dương + âm
- **KHÔNG tóm tắt workflow trong description** — chỉ mô tả khi nào dùng (CSO rule)
- Hơi "pushy": liệt kê nhiều trigger hơn mức tối thiểu để tránh undertrigger
- name: chỉ dùng chữ cái, số, dấu gạch ngang; version: 1.0.0

*Layer 2 — SKILL.md Body:*
- Viết đủ 8 phần (PURPOSE → FINAL CHECK) theo chuẩn
- SKILL.md < 500 dòng; tri thức dài > 30 dòng → tách sang references/
- Dùng imperative form cho instructions ("Đọc...", "Kiểm tra...", "Xuất...")
- Giải thích WHY đằng sau mỗi rule quan trọng thay vì chỉ dùng MUST/NEVER

*Layer 7 — Output Contract:*
- Định nghĩa rõ cấu trúc output, tiêu chí ĐẠT, FINAL CHECK 5–7 mục

**Bước 4: CSO Check (Claude Search Optimization)**
- Kiểm tra description chỉ chứa triggering conditions, không có workflow summary
- Keyword coverage: lỗi cụ thể, triệu chứng, tên tool/command
- Token efficiency: ưu tiên inline cho content < 50 dòng, tách file cho > 100 dòng

**Bước 5: Sinh Examples**
- Tạo thư mục `examples/` với ít nhất:
  - 1 good example: input → process → output rõ ràng
  - 1 anti-example: mẫu xấu + phân tích tại sao sai
- Một example tốt > nhiều example trung bình

**Bước 6: Tự Review nhanh**
- Chạy PHASE 1 (Review) trên skill vừa tạo
- Chấm điểm 6 trục, liệt kê gap nếu có
- Xuất báo cáo ngắn cho user

---

### PHASE 1 — REVIEW (luôn chạy trước khi FIX)

**Bước 1: Khảo sát cấu trúc**
- Đọc toàn bộ thư mục skill (view directory tree)
- Liệt kê tất cả file và folder
- Đối chiếu với cấu trúc chuẩn:
  ```
  skill-name/
  ├── SKILL.md          (bắt buộc)
  ├── references/       (khuyến nghị)
  ├── examples/         (khuyến nghị)
  ├── scripts/          (tuỳ chọn)
  └── assets/           (tuỳ chọn)
  ```

**Bước 2: Đọc `references/9-layer-checklist.md`** — nắm tiêu chí chi tiết

**Bước 3: Kiểm tra từng Layer (0→8)**

| Layer | Tên | Kiểm tra gì |
|-------|-----|-------------|
| 0 | Use Case & Trigger Map | Skill giải quyết việc gì, cho ai, trigger dương/âm |
| 1 | Metadata | YAML frontmatter: name, description, version |
| 2 | SKILL.md Body | Đủ 8 phần? Dưới 500 dòng? Workflow rõ? |
| 3 | References | Có thư mục? SKILL.md chỉ rõ khi nào đọc? |
| 4 | Examples | ≥1 good example + ≥1 anti-example? |
| 5 | Scripts | Có script automation nếu cần? |
| 6 | Assets | Template tách riêng khỏi references? |
| 7 | Output Contract | Có output format + tiêu chí đạt + self-check? |
| 8 | Governance | Version, owner, changelog? |

**Bước 4: Chấm điểm 6 trục** — đọc references/scoring-rubric.md

**Bước 5: Xuất báo cáo Review** — theo Output Format bên dưới

---

### PHASE 2 — FIX (chỉ chạy khi chế độ là `fix` hoặc `review+fix`)

**Nguyên tắc Fix:**
- GIỮ NGUYÊN nội dung chuyên môn gốc — chỉ restructure và bổ sung
- Copy skill sang thư mục làm việc trước khi sửa
- Mỗi layer sửa xong → ghi log thay đổi

**Bước 6: Xử lý từng layer theo thứ tự ưu tiên**

Thứ tự fix (quan trọng nhất trước):

1. **Fix Layer 1 — Metadata**
   - Nếu thiếu YAML frontmatter → thêm
   - Nếu description yếu → viết lại theo công thức:
     ```
     This skill should be used when the user asks to '[trigger 1]',
     '[trigger 2]'... Do NOT use for: [trigger âm].
     ```
   - Giữ nguyên name gốc, thêm version nếu thiếu

2. **Fix Layer 2 — SKILL.md Body**
   - Xác định nội dung gốc thuộc phần nào trong 8 phần
   - Restructure theo đúng 8 phần:
     1. PURPOSE (2–3 câu)
     2. WHEN TO USE (trigger dương ✅ + trigger âm ❌)
     3. EXPECTED INPUTS (bắt buộc vs tuỳ chọn)
     4. WORKFLOW (các bước cụ thể, chỉ rõ resource)
     5. OUTPUT FORMAT (cấu trúc, độ dài, mẫu)
     6. RESOURCE USAGE (khi nào đọc file nào)
     7. GUARDRAILS (điều cấm, giới hạn)
     8. FINAL CHECK (checklist tự kiểm 5–7 mục)
   - Nếu nội dung gốc quá dài → tách phần tri thức sang references/

3. **Fix Layer 7 — Output Contract**
   - Thêm OUTPUT FORMAT nếu thiếu
   - Thêm tiêu chí ĐẠT / CẦN SỬA / KHÔNG ĐẠT
   - Thêm FINAL CHECK checklist

4. **Fix Layer 4 — Examples**
   - Tạo thư mục examples/ nếu chưa có
   - Sinh ít nhất 1 good example từ nội dung skill
   - Sinh ít nhất 1 anti-example với phân tích lỗi
   - Nếu skill gốc đã có ví dụ rời rạc → gom về examples/

5. **Fix Layer 3 — References**
   - Nếu SKILL.md có đoạn tri thức dài (>30 dòng) → tách sang references/
   - Thêm Resource Usage chỉ rõ khi nào đọc file nào

6. **Fix Layer 0, 5, 6, 8** — bổ sung nếu phù hợp với loại skill

**Bước 7: Tổng hợp changelog**
- Liệt kê tất cả thay đổi đã thực hiện
- Ghi version mới (bump minor version)

**Bước 8: Xuất skill đã fix**
- Lưu toàn bộ thư mục skill mới vào /mnt/user-data/outputs/
- Present files cho người dùng

**Bước 9: Chạy lại Review nhanh trên skill đã fix**
- Chấm điểm lại 6 trục
- So sánh trước/sau
- Xuất bảng so sánh điểm

## 5. OUTPUT FORMAT

### Phần 0 — Kết Quả Create

```
# ✨ SKILL MỚI: [tên skill]

## Cấu trúc thư mục
[tree output]

## Files đã tạo
- SKILL.md: [mô tả ngắn]
- examples/good-example.md
- examples/anti-example.md
- references/ (nếu có)

## Review nhanh sau khi tạo
- Điểm: X/30 — [mức]
- Gap cần lưu ý: ...
```

### Phần A — Báo Cáo Review

```
# 📋 BÁO CÁO REVIEW: [tên skill]

## Tổng quan
- Tên: ... | Version: ... | Số files: ...
- Đánh giá: X/30 — [Chưa đạt | Cơ bản | Tốt | Trưởng thành]

## Đánh giá từng Layer
| Layer | Tên | Trạng thái | Ghi chú |
|-------|-----|-----------|---------|
| 0 | Use Case | ✅/⚠️/❌ | ... |
| ... | ... | ... | ... |

## Chấm điểm 6 trục
| Trục | Điểm (1–5) | Nhận xét |
|------|-----------|---------|

## Top 3–5 việc cần sửa
1. [Hành động] — [Layer] — [Ưu tiên]
```

### Phần B — Báo Cáo Fix (chỉ khi có fix)

```
# 🔧 BÁO CÁO FIX: [tên skill]

## Changelog
| # | Thay đổi | Layer | Chi tiết |
|---|---------|-------|---------|

## So sánh trước/sau
| Trục | Trước | Sau | Δ |
|------|-------|-----|---|

## Cấu trúc thư mục mới
[tree output]
```

## 6. RESOURCE USAGE
- Luôn đọc trước khi review: `references/9-layer-checklist.md`
- Đọc khi chấm điểm: `references/scoring-rubric.md`
- Đọc khi viết hoặc fix metadata: `references/metadata-formula.md`
- Tham khảo mẫu: `examples/`

## 7. GUARDRAILS

**Khi Create:**
- KHÔNG viết SKILL.md trước khi xác nhận intent với user
- Description chỉ mô tả triggering conditions — KHÔNG tóm tắt workflow (CSO rule)
- Nếu domain phức tạp hoặc cần tool đặc thù → nghiên cứu trước khi viết
- Skill < 500 dòng; tách knowledge dài sang references/

**Khi Fix:**
- KHÔNG xoá nội dung chuyên môn gốc — chỉ restructure và bổ sung
- KHÔNG thay đổi name trừ khi user yêu cầu
- Khi fix: ưu tiên sửa trực tiếp file, hoặc tạo bản mới nếu user muốn giữ gốc

**Chung:**
- Nếu không hiểu domain → hỏi user trước khi viết hoặc fix
- Skill cá nhân: tối thiểu Layer 1 + 2 + 4 + 7 là đủ
- Giọng văn: constructive, cụ thể, tập trung vào hành động; giải thích WHY thay vì chỉ ra lệnh

## 8. FINAL CHECK
☐ Create: Đã xác nhận intent với user trước khi viết?
☐ Create: Description bắt đầu "Use this skill when..." và không chứa workflow summary?
☐ Create: SKILL.md có đủ 8 phần, < 500 dòng?
☐ Create: Đã sinh ít nhất 1 good example + 1 anti-example?
☐ Create: Đã tự review và chấm điểm 6 trục?
☐ Review: Đã kiểm tra đủ 9 layer?
☐ Review: Mỗi layer có trạng thái rõ ràng (✅/⚠️/❌)?
☐ Review: Đã chấm điểm 6 trục với ghi chú?
☐ Review: Top 3–5 khuyến nghị cụ thể và có ưu tiên?
☐ Fix: Đã giữ nguyên nội dung chuyên môn gốc?
☐ Fix: Đã restructure SKILL.md đủ 8 phần?
☐ Fix: Đã tạo examples nếu thiếu?
☐ Fix: Đã ghi changelog và bump version?
☐ Fix: Đã chạy review lại và so sánh trước/sau?
