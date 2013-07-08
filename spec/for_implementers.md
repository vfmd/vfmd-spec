# vfmd for implementers

This spec describes the [vfmd Markdown syntax] in formal terms, and
is intended to be read by someone implementing this spec to parse or
otherwise programmatically interpret vfmd input. If you only intend
to write a document using vfmd, see [vfmd for writers].

[vfmd Markdown syntax]: introduction.md
[vfmd for writers]: for_writers.md

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

### Block-elements

The _document_ can be seen as a sequence of _block-elements_, where each
_block-element_ consists of a sequence of one or more _lines_.

There are different types of _block-elements_: 

- Paragraph
- Header
- Blockquote
- List
- Code block
- Horizontal rule
- Null block

To identify the _block-elements_ in the _document_, the _document_ is
seen as a sequence of _lines_.

## Identifying block-elements

Given a sequence of _lines_, henceforth called the **input line
sequence**, we need to identify the block-elements in the input,
identify the type of the block-elements and identify which sub-sequence
of lines in the input correspond to which block-element.

The line at which a block-element begins is called a **block-element
start line**. The line at which a block-element ends is called a
**block-element end line**. The sub-sequence of lines starting from the
_block-element start line_ and ending at the _block-element end line_,
both inclusive, constitute the **block-element line sequence**.

Every _block-element end line_ in the _input line sequence_ is
immediately followed by a _block-element start line_, unless the
_block-element end line_ is the last line in the _input line sequence_.
The first line in the _input line sequence_ is a _block-element start
line_, and the last line in the _input line sequence_ is a
_block-element end line_.

The type of the block-element is determined based on the _block-element
start line_. Also, where the block should end (i.e. the corresponding
_block-element end line_) is also determined base on the _block-element
start line_.

