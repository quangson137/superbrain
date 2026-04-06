# Metadata Formula — Cách Viết Description Chuẩn

## Công thức

```yaml
description: >
  Use this skill when the user asks to '[trigger phrase 1]',
  '[trigger phrase 2]', '[trigger phrase 3]'... or needs [outcome].
  Also use when [context trigger].
  Do NOT use for: [anti-trigger 1], [anti-trigger 2].
```

## Nguyên tắc
- 50–100 từ
- Dùng ngôn ngữ người dùng THẬT (không dùng jargon nội bộ)
- Liệt kê 3–6 trigger phrases cụ thể
- Bao gồm cả tiếng Việt lẫn tiếng Anh nếu user base đa ngôn ngữ
- Luôn có ít nhất 1 trigger âm
- Hơi "pushy" — liệt kê nhiều tình huống hơn mức tối thiểu để tránh undertrigger

## Ví dụ yếu → mạnh

❌ Yếu:
```yaml
description: Helps with documents and reports.
```

⚠️ Trung bình:
```yaml
description: Create professional Word documents with formatting.
```

✅ Mạnh:
```yaml
description: >
  Use this skill when the user asks to 'create a Word doc',
  'write a report', 'make a .docx', 'format my document',
  or needs a professional document with headings, tables,
  or page numbers. Also use when editing existing .docx files.
  Do NOT use for: PDFs, spreadsheets, Google Docs, or plain text.
```

## Checklist trước khi hoàn thành
- [ ] Có ≥3 trigger phrases trong dấu nháy đơn?
- [ ] Có "Do NOT use for" với ≥1 anti-trigger?
- [ ] Trong khoảng 50–100 từ?
- [ ] Dùng ngôn ngữ người dùng thật (không jargon)?
