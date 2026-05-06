# Chinese Translation Design

## Scope

Translate the primary public reading path for this repository into Simplified Chinese:

- `README.md`
- canonical Markdown files in `content/`

Do not translate workshop material, legacy one-line redirect files, generated package content, or image files in this phase.

## Structure

Add parallel Chinese files without replacing the English source:

- `README.zh-CN.md`
- `content/zh-CN/*.md`

Keep source file slugs in English so every Chinese file maps directly to the corresponding English file.

## Linking

Chinese internal links point to Chinese files using relative links. External links remain unchanged. Image URLs remain unchanged; only surrounding text and alt text are translated.

## Translation Style

Use natural technical Chinese aimed at engineers. Keep key ecosystem terms in English where useful, especially on first mention. Preserve the author's direct voice and technical judgment, but avoid awkward literal translations of English idioms.

Code identifiers, package names, commands, API fields, JSON keys, and file paths remain unchanged. Explanatory comments and natural-language examples may be translated when they do not alter semantics.

## Attribution

Translated files attribute the work to `0bipinnata0`, identify the original source path, and record baseline commit `d20c728`. Content and images remain under CC BY-SA 4.0; code remains under Apache 2.0.

## Verification

After translation, verify:

- expected Chinese file set exists
- old redirect files were not copied
- Chinese internal links do not point back to English `content/*.md`
- Markdown structure remains readable
- git diff is limited to translation-related files
