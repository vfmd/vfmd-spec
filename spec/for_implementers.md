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

A **string** is a sequence of zero or more _characters_.

**Trimming** a _string_ means removing any leading or trailing
_whitespace_ characters from the _string_. For example, _trimming_
<code>&nbsp;&nbsp;&nbsp;yellow&nbsp;&nbsp;</code> yields `yellow`;
_trimming_ <code>green&nbsp;&nbsp;</code> yields `green`. _Trimming_ a
_string_ that does  not have any leading or trailing _spaces_ has no
effect on the _string_. Trimming a _string_ that is entirely composed of
_whitespace_ yields an empty (zero-length) _string_.
  
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
- Header (atx-style, or setext-style)
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

The type and extent of the _block-element_ is determined as follows:

 1. If the _block-element start line_ is a _blank line_, then the body
    element is of type **null**. The same line is the _block-element end
    line_.

 2. If the leftmost character of the _block-element start line_ is a `#`
    character, it signifies the start of a block-element of type
    **atx-style header**.  The same line is the _block-element end line_.

 3. If the _block-element start line_ is not a _blank line_, and begins
    with four or more consecutive _space_ characters, it signifies the
    start of a block-element of type **code block**. The _block-element
    end line_ is the next subsequent line in the _input line sequence_
    that is immediately succeeded by a succeeding line that satisfies
    one of the following conditions:

     1. The succeeding line is not a _blank line_, and it does not
        begin with four or more consecutive _space_ characters (or)
     2. The succeeding line is a _blank line_, and is immediately
        followed by a line that does not begin with four or more
        consecutive _space_ characters

    If no such _block-element end line_ is found, the last line in the
    _input line sequence_ is the _block-element end line_.

 4. If none of the above conditions apply, and if the _block-element
    start line_ does not start with a _space_ character, and is not the
    last line in the _input line sequence_, and is immediately followed
    by a succeeding line that satisfies at least one of the following
    conditions:

     1. The succeeding line begins with a `-` character, and is composed
        entirely of instances of only the `-` character and optional
        _space_ characters (or)
     2. The succeeding line begins with a `=` character, and is composed
        entirely of instances of only the `=` character and optional
        _space_ characters

    then the block-element that starts at the _block-element start line_
    is said to be of type **setext-style header**, and the succeeding
    line is said to be the _block-element end line_.

Each _block-element line sequence_ has to be processed based on the type
of the block-element. The respective sections for each type of
block-element, given below, discuss that in detail.

### atx-style header

An atx-style header can be formed from a single _line_ that starts with
a `#` character.

The _line_ that constitutes the atx-style header shall match one of the
following regular expression patterns:

 1. With header text: `/^(#+)(.*[^#])#*$/`

    Examples:  

        ## Subheading 1
        ### Third-level heading
        ####Fourth-level####
        ##   Subheading #2   ####
        ###### Six hashes
        ####### Seven hashes
        ######## Eight '#'es

    The length of the matching substring for the first paranthesized
    subexpression is the heading level, subject to a maximum of 6.

    The matching substring for the second paranthesized subexpression
    shall be _trimmed_ to give the header text.

    For example, the HTML outputs for the above expressions are:

        <h2>Subheading 1</h2>
        <h3>Third-level heading</h3>
        <h4>Fourth-level</h4>
        <h2>Subheading #2</h2>
        <h6>Six hashes</h6>
        <h6>Seven hashes</h6>
        <h6>Eight '#'es</h6>

 2. Without header text: `/^(#+)$/`

    The length of the matching substring for the first paranthesized
    subexpression is the heading level, subject to a maximum of 6.
    The header text is empty.

    For example, if the complete _line_ reads:

        #####

    then the corresponding HTML output shall be:

        <h5></h5>

### setext-style header

The  _block-element line sequence_ for a setext-style header shall have
exactly two lines, with the second line beginning with either a `-`
character or a `=` character.

The first line shall be _trimmed_ to give the header text.

If the second line starts with the `=` character, the heading level
shall be 1. If the second line starts with the `-` character, the
heading level shall be 2. No other heading levels are possible in a
setext-style header.

For example, consider the following pairs of _lines_:

    Level One
    =========

    Another Level One
    ======= ===== ===

    Level   Two
    -----------

    Another level two
    -- -- -- -- -- --

The corresponding HTML outputs for the above lines are:

    <h1>Level One</h1>

    <h1>Another Level One</h1>

    <h2>Level   Two</h2>

    <h2>Another level two</h2>

### code block

A code block element can be formed by a sequence of _lines_, where
each _line_ in the sequence is either a _blank line_ or a _line_
beginning with four or more _space_ characters.

For each line in the sequence of _lines_, the leading four _space_
characters are removed, if present, and the resulting sequence of
_lines_, separated by _line breaks_, forms the content of the code
block.

For example, if the following sequence of _lines_ form the code block
element:

        int main() {
            return 42;
        }

The corresponding HTML output shall be:

    <pre><code>int main() {
        return 42;
    }
    </code></pre>

### null block

A null block element can be formed from a single _blank line_. A null
block does not result in any output at all.

