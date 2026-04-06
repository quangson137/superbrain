# Anti-Example: Common Mistakes When Fixing a Skill

## ❌ Mistake 1: Deleting original content
The original skill contained domain-specific guidance. During the fix, it was deleted
and replaced with an empty template. → Domain knowledge is lost.

**Correct**: Preserve the content, just restructure it into the right section.

## ❌ Mistake 2: Renaming the skill
The original skill was named `crm-email-followup`. During the fix, it was renamed
to `professional-email-writer`. → Breaks existing references if users already depend on it.

**Correct**: Keep the original name, only improve the description.

## ❌ Mistake 3: Fabricating domain content
A skill described an internal company process. During the fix, additional
steps that weren't in the original were invented. → Introduces incorrect information.

**Correct**: Only generate structure (headings, checklists, formatting).
For new domain content → ask the user.

## ❌ Mistake 4: Editing the original file directly
The original file was in a read-only directory. Editing it directly → error.

**Correct**: Work in a writable location, then present the output to the user.
