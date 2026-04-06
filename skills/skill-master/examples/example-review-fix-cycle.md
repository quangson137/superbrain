# Ví dụ: Review + Fix một skill sơ khai

## INPUT — Skill gốc (chỉ có 1 file)

```markdown
---
name: email-skill
description: This skill helps with emails and marketing.
---
# Email Skill
Write professional emails for business use.
When user asks for email, write one that is professional and clear.
Use formal tone. Include subject line.
```

## OUTPUT — Phần A: Báo cáo Review

# 📋 BÁO CÁO REVIEW: email-skill

## Tổng quan
- Tên: email-skill | Version: không có | Số files: 1
- Đánh giá: 7/30 — Chưa đạt

## Đánh giá từng Layer
| Layer | Tên | Trạng thái | Ghi chú |
|-------|-----|-----------|---------|
| 0 | Use Case | ❌ | Quá rộng: "emails and marketing" |
| 1 | Metadata | ❌ | Description mơ hồ, không trigger, không version |
| 2 | SKILL.md | ❌ | Chỉ 3 dòng, thiếu 7/8 phần |
| 3 | References | ❌ | Không có |
| 4 | Examples | ❌ | Không có |
| 5 | Scripts | ⬜ | Không bắt buộc |
| 6 | Assets | ⬜ | Không bắt buộc |
| 7 | Output Contract | ❌ | Không có |
| 8 | Governance | ❌ | Không version |

## Chấm điểm 6 trục
| Trục | Điểm | Nhận xét |
|------|-------|---------|
| Chiến lược | 2/5 | "emails and marketing" quá rộng |
| Kích hoạt | 1/5 | Sẽ trigger sai liên tục |
| Vận hành | 1/5 | Không có workflow |
| Chất lượng | 1/5 | Không có output contract |
| Tài nguyên | 1/5 | Không resources |
| Quản trị | 1/5 | Không governance |

## Top 3 việc cần sửa
1. Viết lại metadata — Layer 1 — Khẩn cấp
2. Restructure SKILL.md đủ 8 phần — Layer 2 — Khẩn cấp
3. Tạo ≥2 examples — Layer 4 — Cao

---

## OUTPUT — Phần B: Báo cáo Fix

# 🔧 BÁO CÁO FIX: email-skill

## Changelog
| # | Thay đổi | Layer | Chi tiết |
|---|---------|-------|---------|
| 1 | Viết lại metadata | 1 | Thêm triggers, trigger âm, version |
| 2 | Restructure SKILL.md | 2 | Tách thành 8 phần chuẩn |
| 3 | Thêm output contract | 7 | Format + tiêu chí + final check |
| 4 | Tạo examples/ | 4 | 1 good + 1 anti-example |
| 5 | Bump version | 8 | → 1.0.0 |

## So sánh trước/sau
| Trục | Trước | Sau | Δ |
|------|-------|-----|---|
| Chiến lược | 2 | 4 | +2 |
| Kích hoạt | 1 | 4 | +3 |
| Vận hành | 1 | 4 | +3 |
| Chất lượng | 1 | 4 | +3 |
| Tài nguyên | 1 | 3 | +2 |
| Quản trị | 1 | 3 | +2 |
| **Tổng** | **7** | **22** | **+15** |

Mức đánh giá: Chưa đạt → Tốt
