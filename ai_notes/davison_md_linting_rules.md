# Markdownlint Rules Reference for AI Agents

**CRITICAL**: Follow these rules when creating Markdown documents to avoid linting violations.

## Headings

- **MD001**: Increment heading levels by one only (# → ## → ###, not # → ###)
- **MD003**: Use consistent heading style (ATX `#` or Setext `===`)
- **MD018**: Always add space after `#` in headings (`# Title` not `#Title`)
- **MD019**: Use only one space after `#` in headings
- **MD020**: Add spaces inside closed ATX headings (`# Title #` not `#Title#`)
- **MD021**: Use only one space inside closed ATX headings
- **MD022**: Surround headings with blank lines
- **MD023**: Start headings at beginning of line (no indentation)
- **MD024**: Avoid duplicate heading content
- **MD025**: Use only one top-level heading (H1) per document
- **MD026**: Remove trailing punctuation from headings
- **MD041**: Start files with a top-level heading

## Lists

- **MD004**: Use consistent unordered list style (`-`, `*`, or `+`)
- **MD005**: Use consistent indentation for same-level list items
- **MD007**: Indent unordered lists consistently (default: 2 spaces)
- **MD029**: Use consistent ordered list numbering (1. 2. 3. or 1. 1. 1.)
- **MD030**: Use consistent spaces after list markers
- **MD032**: Surround lists with blank lines

## Code

- **MD014**: Don't use `$` before commands unless showing output
- **MD031**: Surround fenced code blocks with blank lines
- **MD038**: Remove spaces inside code spans (`` `code` `` not `` ` code ` ``)
- **MD040**: Specify language for fenced code blocks (```javascript not```)
- **MD046**: Use consistent code block style (fenced or indented)
- **MD048**: Use consistent code fence style (``` or ~~~)

## Links & Images

- **MD011**: Use correct link syntax `[text](url)` not `(text)[url]`
- **MD034**: Wrap bare URLs in angle brackets `<url>` or use proper link syntax
- **MD039**: Remove spaces inside link text `[text](url)` not `[ text ](url)`
- **MD042**: Ensure links have content `[text](url)` not `[](url)`
- **MD045**: Add alt text to images `![alt text](image.png)`
- **MD051**: Ensure link fragments are valid
- **MD052**: Define all reference links used
- **MD053**: Remove unused reference link definitions
- **MD054**: Use consistent link/image style
- **MD059**: Use descriptive link text (not "click here")

## Whitespace & Formatting

- **MD009**: Remove trailing spaces from lines
- **MD010**: Use spaces instead of hard tabs
- **MD012**: Use only single blank lines (no multiple consecutive blank lines)
- **MD013**: Keep lines under 80 characters (configurable)
- **MD027**: Use only one space after blockquote symbol `> text` not `>  text`
- **MD028**: Don't use blank lines inside blockquotes
- **MD037**: Remove spaces inside emphasis markers `**bold**` not `** bold **`
- **MD047**: End files with single newline character

## Tables

- **MD055**: Use consistent table pipe style
- **MD056**: Ensure consistent column count in tables
- **MD058**: Surround tables with blank lines
- **MD060**: Use consistent table column alignment

## Other

- **MD033**: Avoid inline HTML (use Markdown alternatives)
- **MD035**: Use consistent horizontal rule style (`---`, `***`, or `___`)
- **MD036**: Use headings instead of emphasis for titles
- **MD043**: Follow required heading structure (if configured)
- **MD044**: Use correct capitalization for proper names

## Quick Checklist for AI Agents

1. ✅ Start with H1 heading
2. ✅ Use single spaces after `#` in headings
3. ✅ Surround headings, lists, code blocks, tables with blank lines
4. ✅ Specify language for code blocks
5. ✅ Add alt text to images
6. ✅ Use descriptive link text
7. ✅ Remove trailing spaces
8. ✅ End file with single newline
9. ✅ Keep lines under 80 characters
10. ✅ Use consistent list markers and indentation
