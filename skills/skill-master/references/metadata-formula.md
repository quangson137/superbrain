# Metadata Formula — How to Write a Good Description

## Formula

```yaml
description: >
  Use this skill when the user asks to '[trigger phrase 1]',
  '[trigger phrase 2]', '[trigger phrase 3]'... or needs [outcome].
  Also use when [context trigger].
  Do NOT use for: [anti-trigger 1], [anti-trigger 2].
```

## Principles
- 50–100 words
- Use real user language (no internal jargon)
- List 3–6 specific trigger phrases
- Include both Vietnamese and English triggers if the user base is multilingual
- Always include at least 1 negative trigger
- Be slightly "pushy" — list more scenarios than the minimum to avoid undertriggering

## Weak → Strong examples

❌ Weak:
```yaml
description: Helps with documents and reports.
```

⚠️ Mediocre:
```yaml
description: Create professional Word documents with formatting.
```

✅ Strong:
```yaml
description: >
  Use this skill when the user asks to 'create a Word doc',
  'write a report', 'make a .docx', 'format my document',
  or needs a professional document with headings, tables,
  or page numbers. Also use when editing existing .docx files.
  Do NOT use for: PDFs, spreadsheets, Google Docs, or plain text.
```

## Pre-completion checklist
- [ ] ≥3 trigger phrases in single quotes?
- [ ] "Do NOT use for" with ≥1 negative trigger?
- [ ] 50–100 words?
- [ ] Real user language (no jargon)?
