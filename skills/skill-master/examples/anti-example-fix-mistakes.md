# Anti-Example: Những Lỗi Khi Fix Skill

## ❌ Lỗi 1: Xoá nội dung gốc
Skill gốc có đoạn hướng dẫn domain-specific. Khi fix, đã xoá mất
và thay bằng template trống. → Mất tri thức chuyên môn.

**Đúng**: Giữ nguyên nội dung, chỉ restructure vào đúng phần.

## ❌ Lỗi 2: Đổi tên skill
Skill gốc tên `crm-email-followup`. Khi fix, đổi thành
`professional-email-writer`. → Phá vỡ liên kết nếu đã có user dùng.

**Đúng**: Giữ nguyên name, chỉ cải thiện description.

## ❌ Lỗi 3: Bịa nội dung chuyên môn
Skill về quy trình nội bộ công ty. Khi fix, tự sinh thêm
quy trình không có trong skill gốc. → Sai thông tin.

**Đúng**: Chỉ sinh cấu trúc (headings, checklist, format).
Nội dung chuyên môn mới → hỏi người dùng.

## ❌ Lỗi 4: Sửa trực tiếp file gốc
File gốc nằm trong thư mục read-only. Sửa trực tiếp → lỗi.

**Đúng**: Copy sang /home/claude/, sửa ở đó, xuất ra /mnt/user-data/outputs/.
