# vfmd for implementers

This document describes the [vfmd Markdown syntax] in formal terms, and
is intended to be read by someone implementing this spec to parse or
otherwise programmatically interpret vfmd input.

[vfmd Markdown syntax]: introduction.md

This document is organized as follows: We start by defining a few terms
that shall be used in the rest of this spec. Then, we discuss how
block-level vfmd elements may be identified, and then, we discuss how
span-level vfmd elements within a block-level element may be identified.

## Definitions

### Document

The input Markdown text is called the **document**.

The _document_ contains Unicode text in UTF-8 encoding without any
leading Byte-Order-Mark. Any byte sequences in the input that are
invalid in UTF-8 encoding are filtered off and are ignored. Therefore,
for the following discussion, the _document_ is considered to not have
any invalid byte sequences.

### Characters

A **character** is an atomic unit of text specified as a Unicode code
point and encoded in UTF-8 encoding.

The _document_ consists of a sequence of _characters_, where the
_characters_ may represent either markup or character data.

A `#x09 (TAB)` _character_ in the input shall be treated as four
consecutive `#x20 (SPACE)` characters.

A `#x20 (SPACE)` character is henceforth called a **space** character.

The character sequence `#x0D #x0A (CRLF)` in the input shall be treated
as a single `#x0A (LF)` character.

A `#x0A (LF)` character is henceforth called a **line break**.

A contiguous sequence of one or more _space_ or _line break_ characters
constitutes **whitespace**.

### Lines

When we split the _document_ on _line breaks_, we get **lines**. The
_document_ can then be seen as a sequence of _lines_, separated by _line
breaks_.

If a _line_ contains no _characters_, or if all _characters_ in the
_line_ are _space_ characters, that line is called a **blank line**.


