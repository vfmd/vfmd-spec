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
Any _character_ that is not a `#x20 (SPACE)` character is henceforth
called a **non-space** character.

The character sequence `#x0D #x0A (CRLF)` in the input shall be treated
as a single `#x0A (LF)` character.

A `#x0A (LF)` character is henceforth called a **line break**.

A **whitespace** character is one of the following characters: `#x09
(TAB)`, `#x0A (LF)`, `#x0C (FF)`, `#x0D (CR)` or `#x20 (SPACE)`.

A **string** is a sequence of zero or more _characters_.

**Trimming** a _string_ means removing any leading or trailing
_whitespace_ characters from the _string_. For example, _trimming_ <code
style="white-space: pre;">   yellow  </code> yields `yellow`; _trimming_
<code style="white-space: pre;">green  </code> yields `green`.
_Trimming_ a _string_ that does  not have any leading or trailing
_spaces_ has no effect on the _string_. Trimming a _string_ that is
entirely composed of _whitespace_ characters yields an empty
(zero-length) _string_.

**Simplifying** a _string_ means _trimming_ the string, and in addition,
replacing each sequence of internal _whitespace_ characters in the
_string_ with a single _space_ character. For example, _simplifying_ <code
style="white-space: pre;">   Amazing   Maurice  </code> yields `Amazing Maurice`; _simplifying_
<code style="white-space: pre;">educated   rodents  </code> yields
`educated rodents`.

**Escaping** a _character_ in a string means placing a `\` (backslash)
just before the _character_ in the string, where the `\` used for
escaping remains unescaped (i.e. the escaping `\` shall not be preceded
by an unescaped `\`).

A **quoted string** is a _string_ that consists at least two
characters and either

 1. begins with an unescaped `'` (single quote) character, and ends with
    an unescaped `'` character, and does not contain any other instance
    of an unescaped `'` character

    or

 2. begins with an unescaped `"` (double quote) character, and ends with
    an unescaped `"` character, and does not contain any other instance
    of an unescaped `"` character

The substring of the _quoted string_ that excludes the first and the
last character of the _quoted string_ is called the _enclosed string_ of
the _quoted string_.

### Lines

When we split the _document_ on _line breaks_, we get **lines**. The
_document_ can then be seen as a sequence of _lines_, separated by _line
breaks_.

If a _line_ contains no _characters_, or if all _characters_ in the
_line_ are _space_ characters, that line is called a **blank line**.

### Block-elements

The _document_ can be seen as a sequence of block-elements, where each
block-element consists of a sequence of one or more _lines_.

There are different types of block-elements: 

- Paragraph
- Header (atx-style, or setext-style)
- Blockquote
- List
- Code block
- Horizontal rule
- Null block

To identify the block-elements in the _document_, the _document_ is
seen as a sequence of _lines_.

## Identifying block-elements

Given a sequence of _lines_, henceforth called the **input line
sequence**, we need to identify the block-elements in the input,
identify the type of the block-elements and identify which sub-sequence
of lines in the input correspond to which block-element.

### The block-element line sequence

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

The idea is to break the _input line sequence_ into a series of
_block-element line sequences_. Each _block-element line sequence_ shall
consist of only the _lines_ that correspond to a particular
block-element.

### Type and extent of a block-element

The type of the block-element is determined based on the _block-element
start line_ and, in some cases, the line following the _block-element
start line_.  The line at which the block should end (i.e.  the
corresponding _block-element end line_) is determined based on the
_block-element start line_ and subsequent lines.

**Definitions:** The regular expression pattern `/^( *[\*\-\+] +)[^ ]/`
is called the **unordered list starter pattern**. The regular expression
pattern `/^( *([0-9]+)\. +)[^ ]/` is called the **ordered list starter
pattern**.

The following rules are to be followed in determining the type of the
block-element and the _block-element end line_:

 1. If the _block-element start line_ is a _blank line_, then the body
    element is of type **null**. The same line is the _block-element end
    line_.
 
 2. If the _block-element start line_ does not begin with four or more
    consecutive _space_ characters, and if the _block-element start
    line_ matches the regular expression pattern
    `/^ *\[([^\\\[\]]|\\.)*\] *: *([^ <>]+|<[^<>]*>)/`, then the
    block-element is of type **reference-resolution block**. The same
    line is the _block-element end line_.

 3. If none of the above conditions apply, and if the _block-element
    start line_ is not the last line in the _input line sequence_, and
    is immediately followed by a succeeding line that matches the
    regular expression pattern `/^(-+|=+) *$/`, then the block-element
    is said to be of type **setext-style header**, and the succeeding
    line is said to be the _block-element end line_.

 4. If none of the above conditions apply, and if the _block-element
    start line_ begins with four or more consecutive _space_ characters,
    it signifies the start of a block-element of type **code block**.
    The _block-element end line_ is the next subsequent line in the
    _input line sequence_, starting from and inclusive of the
    _block-element start line_, that is immediately succeeded by a
    succeeding line that satisfies one of the following conditions:

     1. The succeeding line is not a _blank line_, and it does not
        begin with four or more consecutive _space_ characters (or)
     2. The succeeding line is a _blank line_, and is immediately
        followed by a line that does not begin with four or more
        consecutive _space_ characters

    If no such _block-element end line_ is found, the last line in the
    _input line sequence_ is the _block-element end line_.

 5. If none of the above conditions apply, and if the first character
    of the _block-element start line_ is a `#` character, it signifies
    the start of a block-element of type **atx-style header**.  The same
    line is the _block-element end line_.

 6. If none of the above conditions apply, and if the first
    _non-space_ character in the _block-element start line_ is a `>`
    character, then the block-element is of type **blockquote**. The
    _block-element end line_ is the next subsequent line in the _input
    line sequence_ that is a _blank line_ and is immediately succeeded
    by a succeeding line that satisfies one of the following conditions:

     1. The succeeding line is a _blank line_ (or)
     2. The succeeding line begins with four or more consecutive _space_
        characters (or)
     3. The first _non-space_ character in the succeeding line is not
        a `>` character
    
    If no such _block-element end line_ is found, the last line in the
    _input line sequence_ is the _block-element end line_.

 7. If none of the above conditions apply, and if the _block-element
    start line_ contains three or more `*` characters, and is composed
    entirely of instances of the `*` character and optional _space_
    characters, then the line forms a block-element of type **horizontal
    rule**.

    Similarly, if the _block-element start line_ contains either three
    or more `-` characters, or three or more `_` characters, and is
    composed entirely of the same character and optional _space_
    characters, then the block-element is of type _horizontal rule_.

    The _block-element end line_ is the same as the _block-element start
    line_.

 8. If none of the conditions specified above apply, and if the
    _block-element start line_ matches the _unordered list starter
    pattern_ (i.e. the regular expression `/^( *[\*\-\+] +)[^ ]/`) then
    the block-element is of type **unordered list**. The matching
    substring for the first and only parenthesized subexpression in the
    pattern is called the _unordered list starter string_. The number of
    _characters_ in the _unordered list starter string_ is called the
    _unordered-list-starter-string-length_.

    For example, consider the following _block-element start line_
    (which has three _space_ characters, followed by an asterisk,
    followed by two _space_ characters, followed by the word "Peanuts"):
  
           *  Peanuts

    The _unordered list starter string_ in the above example consists of
    the first 6 characters of the line, i.e. the entire part before the
    word "Peanuts". The _unordered-list-starter-string-length_ is 6.
   
    The _block-element end line_ is the next subsequent line in the
    _input line sequence_, starting from and inclusive of the
    _block-element start line_, that satisfies one of the following
    conditions:

     1. The line is a _blank line_ and is immediately succeeded by
        another _blank line_

        (or)

     2. The line is a _blank line_ and is immediately succeeded by a
        succeeding line that satisfies all of the following conditions:

         1. The succeeding line does not start with the _unordered list
            starter string_ seen in the _block-element start line_
         2. The first _unordered-list-starter-string-length_ characters
            of the succeeding line include _non-space_ characters

        (or)

     3. The line is not a _blank line_ and is immediately succeeded by
        a succeeding line that statisfies all of the following
        conditions:

         1. The succeeding line does not start with the _unordered list
            starter string_ seen in the _block-element start line_
         2. The first _unordered-list-starter-string-length_ characters
            of the succeeding line include _non-space_ characters
         3. The succeeding line matches either the _unordered list
            starter pattern_ or the _ordered list starter pattern_

    If no such _block-element end line_ is found, the last line in the
    _input line sequence_ is the _block-element end line_.

 9. If none of the conditions specified above apply, and if the
    _block-element start line_ matches the _ordered list starter
    pattern_ (i.e. the regular expression `/^( *([0-9]+)\. +)[^ ]/`)
    then the block-element is of type **ordered list**. The length of
    the matching substring for the first (i.e. inner) parenthesized
    subexpression in the pattern is called the
    _ordered-list-starter-string-length_.
    
    For example, consider the following _block-element start line_
    (which has three _space_ characters, followed by the number '1',
    followed by a dot, followed by two _space_ characters, followed by
    the word "Peanuts"):
  
           1.  Peanuts

    The _ordered-list-starter-string-length_ in the above example is 7.
   
    The _block-element end line_ is the next subsequent line in the
    _input line sequence_, starting from and inclusive of the
    _block-element start line_, that satisfies one of the following
    conditions:

     1. The line is a _blank line_ and is immediately succeeded by
        another _blank line_

        (or)

     2. The line is a _blank line_ and is immediately succeeded by a
        succeeding line that satisfies all of the following conditions:

         1. The succeeding line does not match the _ordered list starter
            pattern_
         2. The first _ordered-list-starter-string-length_ characters of
            the succeeding line include _non-space_ characters

        (or)

     3. The line is not a _blank line_ and is immediately succeeded by
        a succeeding line that statisfies all of the following
        conditions:

         1. The succeeding line does not match the _ordered list starter
            pattern_
         2. The first _ordered-list-starter-string-length_ characters of
            the succeeding line include _non-space_ characters
         3. The succeeding line matches the _unordered list starter
            pattern_

    If no such _block-element end line_ is found, the last line in the
    _input line sequence_ is the _block-element end line_.

10. If none of the above conditions apply, then the block-element is of
    type **paragraph**.

    In order to find the _block-element end line_, we need to make use
    of a HTML parser. To the HTML parser, we feed the characters of each
    line, starting from the _block-element start line_. After feeding
    all characters of every line, we feed a `#x0A (LF)` character (i.e.
    a _line break_) to the HTML parser, and observe the state of the
    HTML parser.

    Of the the many possible states of a HTML parser, the following is
    the list of states that we are interested in:

     1. Within a HTML tag (open / close / self-closing tag)
     2. Within the contents of a HTML element, but not within a HTML
        open or close tag
     3. Within a HTML comment
     4. Outside of any HTML element or comment
 
    The _block-element end line_ is the next subsequent _line_ in the
    _input line sequence_, starting from and inclusive of the
    _block-element start line_, that satisfies all the following
    conditions:

    <!-- For some reason, Redcarpet requires a comment here to
         correctly display the following list -->

     1. At the end of feeding the _line_ and a _line break_ to the HTML
        parser, all the following conditions are satisfied:

         1. The HTML parser state is not "within a HTML tag"

         2. The HTML parser state is not "within a HTML comment"

         3. If the HTML parser state is "within the contents of a HTML
            element", then the containing HTML element or any of its
            ancestor elements is not one of the following HTML elements:
            `pre`, `script`, `style`

     2. The _line_ is a _blank line_, or is immediately succeeded by a
        succeeding line that satisfies at least one of the following
        conditions:

         1. The leftmost _non-space_ character in the succeeding line is
            a `>` character

            (or)

         2. The succeeding line contains three or more `*` characters,
            and is composed entirely of instances of the `*` character
            and optional _space_ characters (or similarly with `-` or
            `_` characters)


Using the above rules, the _input line sequence_ is broken down into a
series of _block-element line sequences_, and the block-element type of
each _block-element line sequence_ is identified.

## Interpreting block-elements

This section assumes that the _input line sequence_ has been broken down
into a series of _block-element line sequences_, and that the
block-element type of each _block-element line sequence_ has been
identified. For a given block-element type, the procedure to interpret a
_block-element line sequence_ is discussed in this section.

### atx-style header

The  _block-element line sequence_ for an atx-style header shall have a
single line that starts with a `#` character.

The single _line_ in the _block-element line sequence_ shall match one
of the following regular expression patterns:

 1. With header text: `/^(#+)(.*[^#])#*$/`

    Examples:  

        ## Subheading 1
        ### Third-level *heading*
        ####Fourth-level####
        ##   Subheading #2   ####
        ###### Six hashes
        ####### Seven hashes
        ######## Eight '#'es

    The length of the matching substring for the first parenthesized
    subexpression is the heading level, subject to a maximum of 6.

    The matching substring for the second parenthesized subexpression
    shall be _trimmed_ to give a _header text run_. The result of
    processing the _header text run_ as a _text run_ shall form the
    content of the header element.

    For example, the HTML outputs for the above expressions are:

        <h2>Subheading 1</h2>
        <h3>Third-level <em>heading</em></h3>
        <h4>Fourth-level</h4>
        <h2>Subheading #2</h2>
        <h6>Six hashes</h6>
        <h6>Seven hashes</h6>
        <h6>Eight '#'es</h6>

 2. Without header text: `/^(#+)$/`

    The length of the matching substring for the first parenthesized
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

The first line shall be _trimmed_ to give a _header text run_. The
result of processing the _header text run_ as a _text run_ shall form
the content of the header element.

If the second line starts with the `=` character, the heading level
shall be 1. If the second line starts with the `-` character, the
heading level shall be 2. No other heading levels are possible in a
setext-style header.

For example, consider the following pairs of _lines_:

    Level One
    =========

    Another *Level One*
    ===================

    Level   Two
    -----------

    Another level two
    -----------------

The corresponding HTML outputs for the above lines are:

    <h1>Level One</h1>

    <h1>Another <em>Level One</em></h1>

    <h2>Level   Two</h2>

    <h2>Another level two</h2>

### code block

Each line in the  _block-element line sequence_ for a code block element
shall either be a _blank line_ or a _line_ beginning with four or more
_space_ characters.

For each line in the _block-element line sequence_, the leading four
_space_ characters are removed, if present, and the resulting sequence
of _lines_, separated by _line breaks_, forms the content of the code
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

### blockquote

The _block-element line sequence_ for a blockquote element shall
have one or more _lines_, some of which might have the `>` character as
the first _non-space_ character in the _line_.

If the last _line_ in the _block-element line sequence_ is a _blank
line_, the last _line_ is ignored.

Each _line_ in the _block-element line sequence_ is processed to produce
a modified sequence of _lines_, called the _blockquote-processed line
sequence_. The following processing is to be done for each _line_:

 1. If the _line_ matches the regular expression `/^ *> /`, then the
part of the _line_ that matches the said regular expression shall be removed
from the line  
 2. If the pattern in (1) above is not satisfied, and if the _line_ matches
the regular expression `/^ *>/`, then the part of the _line_ that
matches the said regular expression shall be removed from the line

The _blockquote-processed line sequence_ obtained this way can be
considered as the _input line sequence_ for a sequence of block-elements
nested within the blockquote. The result of interpreting that _input
line sequence_ further into block-elements shall form the content of the
blockquote element.

For example, consider the following _block-element line sequence_:

      > In Perl, a Hello World is 
      > written as follows:
      >
      >     print "Hello World!\n";

After processing each line in the above _block-element line sequence_,
the _blockquote-processed line sequence_ obtained is as follows:

    In Perl, a Hello World is 
    written as follows:
    
        print "Hello World!\n";

When we treat the _blockquote-processed line sequence_ as an _input line
sequence_, we can recognize nested block elements in it of type
paragraph and code block. The HTML equivalent for the _lines_ in the
_blockquote-processed line sequence_ is as follows:

    <p>In Perl, a Hello World is 
    written as follows:</p>
    
    <pre><code>print "Hello World!\n";
    </code></pre>

Therefore, the HTML equivalent for the given  _block-element line
sequence_ is:

    <blockquote>
    <p>In Perl, a Hello World is 
    written as follows:</p>
    
    <pre><code>print "Hello World!\n";
    </code></pre>
    </blockquote>

### horizontal rule

The  _block-element line sequence_ for a null block element shall have a
single _line_ that is composed entirely of either `*`, `-` or `_`
characters, along with optional _space_ characters.

The output shall consist of a horizontal rule.

For example:

    Last line of a paragraph.

    * * *

    First line of a paragraph.

The corresponding HTML output shall be:

    <p>Last line of a paragraph.</p>

    <hr/>

    <p>First line of a paragraph.</p>

### unordered list

The _block-element line sequence_ for an unordered list block shall have
one or more _lines_.

The first line in the _block-element line sequence_ would match the
_unordered list starter pattern_ (i.e. the regular expression
`/^( *[\*\-\+] +)[^ ]/`). The matching substring for the first and only
parenthesized subexpression in that pattern is called the _unordered
list starter string_. The number of characters in the _unordered list
starter string_ is called the _unordered-list-starter-string-length_.

We first divide the _block-element line sequence_ into a series of
_unordered list item line sequences_. The lines in a particular
_unordered list item line sequence_ correspond to one list item in the
list.

Every _line_ in the _block-element line sequence_ that starts with the
_unordered list starter string_ is called an _unordered list item start
line_. Each _unordered list item start line_ signifies the beginning of
a new _unordered list item line sequence_. An _unordered list item line
sequence_ consists of the sequence of _lines_ starting from (and
inclusive of) an _unordered list item start line_, and ending at (and
excluding) the next subsequent _unordered list item start line_. If
there is no subsequent _unordered list item start line_, the _unordered
list item line sequence_ ends at the last _line_ of the _block-element
line sequence_.

We have now divided the _block-element line sequence_ into a series of
_unordered list item line sequences_. The first line of each _unordered
list item line sequence_ starts with the _unordered list starter
string_.

Each _line_ in the _unordered list item line sequence_ is processed to
produce a modified sequence of _lines_, called the
_unordered-list-item-processed line sequence_. The following processing
is to be done for each _line_:

 1. If the _line_ is the first line of the _unordered list item line
    sequence_:
    
    The _line_ would start with the _unordered list starter string_.
    The _unordered list starter string_ shall be removed from the
    beginning of the _line_.

 2. If the _line_ is not the first line of the _unordered list item line
    sequence_:
    
    The _line_ would start with zero or more _space_ characters. The
    leading _space_ characters, if any, should be removed as given
    below:

     1. If the number of leading _space_ characters exceeds the
        _unordered-list-starter-string-length_, the number of leading
        _space_ characters removed should be equal to the
        _unordered-list-starter-string-length_.
     2. If the number of leading _space_ characters is less than or
        equal to the _unordered-list-starter-string-length_, all the
        leading _space_ characters should be removed.

The _unordered-list-item-processed line sequence_ obtained this way can
be considered as the _input line sequence_ for a sequence of
block-elements nested within the list item. The result of interpreting
that _input line sequence_ further into block-elements shall form the
content of the list element. 

The list elements so obtained are combined into a sequence to form the
complete unordered list in the output.

For example, consider the following _block-element line sequence_:

    * First item 1

    * Second item 1
    Second item 2

          Code block

    * Third item 1

        * Nested item 1

The _unordered list starter string_ for the above example is an asterisk
followed by a single _space_ character. The
_unordered-list-starter-string-length_ is 2.

The 1<sup>st</sup>, 3<sup>rd</sup> and 8<sup>th</sup> lines in the
_block-element line sequence_ start with the _unordered list starter
string_, and are therefore _unordered list item start lines_. (The
10<sup>th</sup> line does contain the _unordered list starter string_,
but does not start with the _unordered list starter string_, so it's not
an _unordered list item start line_.)  Therefore, there are three
_unordered list item line sequences_ in the above example, as follows:

 1. The _lines_ 1 and 2 form the first _unordered list item
    line sequence_
 2. The _lines_ 3-7 form the second _unordered list item line sequence_
 3. The _lines_ 8-10 form the third and last _unordered list item
    line sequence_

Each _unordered list item line sequence_ is then processed to obtain the
_unordered-list-item-processed line sequence_.  When we treat each
_unordered-list-item-processed line sequence_ as an _input line
sequence_, we can recognize nested block elements in it.

The first _unordered list item line sequence_ looks like:

<pre><code>* First item 1

</code></pre>

To obtain the corresponding _unordered-list-item-processed line
sequence_, we need to remove the _unordered list starter string_ from
the beginning of the first line.

The first _unordered-list-item-processed line sequence_ is therefore:

<pre><code>First item 1

</code></pre>

When this _unordered-list-item-processed line sequence_ is processed as
an _input line sequence_ to identify block-elements in it, we get a
single paragraph block-element.  The corresponding HTML would be:

    <p>First item 1</p>

The second _unordered list item line sequence_ looks like:

<pre><code>* Second item 1
Second item 2

      Code block

</code></pre>

To obtain the corresponding _unordered-list-item-processed line
sequence_, we need to remove the _unordered list starter string_ from
the beginning of the first line, and remove leading _space_ characters,
subject to a maximum of 2 _space_ characters (because the
_unordered-list-starter-string-length_ is 2), from the subsequent lines.

The second _unordered-list-item-processed line sequence_ is therefore:

<pre><code>Second item 1
Second item 2

    Code block

</code></pre>

When this _unordered-list-item-processed line sequence_ is processed as
an _input line sequence_ to identify block-elements in it, we get a
paragraph followed by a code block.  The corresponding HTML would be:

    <p>Second item 1
    Second item 2</p>

    <pre><code>Code block
    </code></pre>

The third _unordered list item line sequence_ looks like:

    * Third item 1

        * Nested item 1

To obtain the corresponding _unordered-list-item-processed line
sequence_, we need to remove the _unordered list starter string_ from
the beginning of the first line, and remove leading _space_ characters,
subject to a maximum of 2 _space_ characters, from the subsequent lines.

The third _unordered-list-item-processed line sequence_ is therefore:

    Third item 1

      * Nested item 1

When this _unordered-list-item-processed line sequence_ is processed as
an _input line sequence_ to identify block-elements in it, we get a
paragraph followed by an unordered list.  The corresponding HTML would
be:

    <p>Third item 1</p>

    <ul>
        <li>Nested item 1</li>
    </ul>

Putting the content of all the list items together, the HTML equivalent
for the complete _block-element line sequence_ of the unordered list in
this example would be:

    <ul>
        <li><p>First item 1</p></li>
        <li><p>Second item 1
        Second item 2</p>

        <pre><code>Code block
        </code></pre></li>
        <li><p>Third item 1</p>

        <ul>
            <li>Nested item 1</li>
        </ul></li>
    <ul>

### ordered list

The _block-element line sequence_ for an ordered list block shall have
one or more _lines_.

The first line in the _block-element line sequence_ would match the
_ordered list starter pattern_ (i.e. the regular expression
`/^( *([0-9]+)\. +)[^ ]/`). The length of the matching substring for the
first (i.e. outer) parenthesized subexpression in the pattern is called
the _ordered-list-starter-string-length_. The matching substring for the
second (i.e. inner) parenthesized subexpression in the pattern is called
the _ordered list starting number_.

We first divide the _block-element line sequence_ into a series of
_ordered list item line sequences_. The lines in a particular
_ordered list item line sequence_ correspond to one list item in the
list.

Every _line_ in the _block-element line sequence_ that satisfies all the
following conditions is called an _ordered list item start line_:

 1. The _line_ matches the _ordered list starter pattern_
 2. The first _ordered-list-starter-string-length_ characters of the
    _line_ include _non-space_ characters

Each _ordered list item start line_ signifies the beginning of a new
_ordered list item line sequence_. An _ordered list item line sequence_
consists of the sequence of _lines_ starting from (and inclusive of) an
_ordered list item start line_, and ending at (and excluding) the next
subsequent _ordered list item start line_. If there is no subsequent
_ordered list item start line_, the _ordered list item line sequence_
ends at the last _line_ of the _block-element line sequence_.

We have now divided the _block-element line sequence_ into a series of
_ordered list item line sequences_. The first line of each _ordered list
item line sequence_ matches the _ordered list starter pattern_.

Each _line_ in the _ordered list item line sequence_ is processed to
produce a modified sequence of _lines_, called the
_ordered-list-item-processed line sequence_. The following processing is
to be done for each _line_:

 1. If the _line_ is the first line of the _ordered list item line
    sequence_:
    
    The _line_ would match the _ordered list starter pattern_. The
    matching substring for the first (i.e. outer) parenthesized
    subexpression in the pattern shall be removed from the beginning of
    the _line_.
 2. If the _line_ is not the first line of the _ordered list item line
    sequence_:
    
    The _line_ would start with zero or more _space_ characters. The
    leading _space_ characters, if any, should be removed as given
    below:

     1. If the number of leading _space_ characters exceeds the
        _ordered-list-starter-string-length_, the number of leading
        _space_ characters removed should be equal to the
        _ordered-list-starter-string-length_.
     2. If the number of leading _space_ characters is less than or
        equal to the _ordered-list-starter-string-length_, all the
        leading _space_ characters should be removed.

The _ordered-list-item-processed line sequence_ obtained this way can be
considered as the _input line sequence_ for a sequence of block-elements
nested within the list item. The result of interpreting that _input line
sequence_ further into block-elements shall form the content of the list
element.

The list elements so obtained are combined into a sequence to form the
complete ordered list in the output. 

The numbering for the ordered list should start from the _ordered list
starting number_. For HTML output, if the _ordered list starting number_
is the number '1', the corresponding `ol` start tag in the output shall
not have the `start` attribute; if the _ordered list starting number_ is
not the number '1', the corresponding `ol` start tag in the output shall
include the `start` attribute with the the _ordered list starting
number_ as the attribute value.

For example, consider the following _block-element line sequence_:

    1. First item 1

    2. Second item 1
    Second item 2

          Code block

    3. Third item 1

        1. Nested item 1

When we match the first line against the _ordered list starter pattern_,
the matching substring for the parenthesized subexpression is obtained
as <code>1. </code> (i.e. the number '1', followed by a dot, followed by
a single _space_ character). The _ordered-list-starter-string-length_ is
therefore 3. Also, the _ordered list starting number_ is identified as
the number '1'.

The 1<sup>st</sup>, 3<sup>rd</sup>, 8<sup>th</sup> and 10<sup>th</sup>
lines in the _block-element line sequence_ match the _ordered list
starter pattern_, but only the 1<sup>st</sup>, 3<sup>rd</sup> and
8<sup>th</sup> lines are such that the first 3 characters of the line
include _non-space_ characters. So only the 1<sup>st</sup>,
3<sup>rd</sup> and 8<sup>th</sup> lines are are _ordered list item start
lines_.  Therefore, there are three _ordered list item line sequences_
in the above example, as follows:

 1. The _lines_ 1 and 2 form the first _ordered list item line sequence_
 2. The _lines_ 3-7 form the second _ordered list item line sequence_
 3. The _lines_ 8-10 form the third and last _ordered list item line
    sequence_

Each _ordered list item line sequence_ is then processed to obtain the
_ordered-list-item-processed line sequence_.  When we treat each
_ordered-list-item-processed line sequence_ as an _input line sequence_,
we can recognize nested block elements in it.

The first _ordered list item line sequence_ looks like:

<pre><code>1. First item 1

</code></pre>

To obtain the corresponding _ordered-list-item-processed line sequence_,
we need to match the line against the _ordered list starter pattern_ and
remove the matching substring for the first parenthesized subexpression.
The matching substring in this case is <code>1. </code> (i.e. the number
'1', followed by a dot, followed by a single space character).

The first _ordered-list-item-processed line sequence_ is therefore:

<pre><code>First item 1

</code></pre>

When this _ordered-list-item-processed line sequence_ is processed as an
_input line sequence_ to identify block-elements in it, we get a single
paragraph block-element.  The corresponding HTML would be:

    <p>First item 1</p>

The second _ordered list item line sequence_ looks like:

<pre><code>2. Second item 1
Second item 2

      Code block

</code></pre>

To obtain the corresponding _ordered-list-item-processed line sequence_,
we need to match the first line against the _ordered list starter
pattern_ and remove the matching substring for the first parenthesized
subexpression. The matching substring in this case is <code>2. </code>
(i.e. the number '2', followed by a dot, followed by a single space
character). From subsequent lines, we need to remove leading _space_
characters, subject to a maximum of 3 _space_ characters (because the
_ordered-list-starter-string-length_ is 3).

The second _ordered-list-item-processed line sequence_ is therefore:

<pre><code>Second item 1
Second item 2

    Code block

</code></pre>

When this _ordered-list-item-processed line sequence_ is processed as
an _input line sequence_ to identify block-elements in it, we get a
paragraph followed by a code block.  The corresponding HTML would be:

    <p>Second item 1
    Second item 2</p>

    <pre><code>Code block
    </code></pre>

The third _ordered list item line sequence_ looks like:

    3. Third item 1

        1. Nested item 1

To obtain the corresponding _ordered-list-item-processed line sequence_,
we need to match the first line against the _ordered list starter
pattern_ and remove the matching substring for the first parenthesized
subexpression. The matching substring in this case is <code>3. </code>
(i.e. the number '3', followed by a dot, followed by a single space
character). From subsequent lines, we need to remove leading _space_
characters, subject to a maximum of 3 _space_ characters (because the
_ordered-list-starter-string-length_ is 3).

The third _ordered-list-item-processed line sequence_ is therefore:

    Third item 1

     1. Nested item 1

When this _ordered-list-item-processed line sequence_ is processed as
an _input line sequence_ to identify block-elements in it, we get a
paragraph followed by an ordered list.  The corresponding HTML would
be:

    <p>Third item 1</p>

    <ol>
        <li>Nested item 1</li>
    </ol>

Putting the content of all the list items together, the HTML equivalent
for the complete _block-element line sequence_ of the ordered list in
this example would be:


    <ol>
        <li><p>First item 1</p></li>
        <li><p>Second item 1
        Second item 2</p>

        <pre><code>Code block
        </code></pre></li>
        <li><p>Third item 1</p>

        <ol>
            <li>Nested item 1</li>
        </ol></li>
    </ol>

### paragraph

The _block-element line sequence_ for a paragraph block shall have
one or more _lines_.

The lines in the _block-element line sequence_ are joined together into
a single sequence of _characters_, with a _line feed_ after each line.
The resulting sequence of _characters_ is called the _paragraph text_.
The result of interpreting the _paragraph text_ as a _text run_ shall
form the content of the paragraph element.

The _paragraph text_ needs to be run through a HTML parser to determine
how the content of the paragrah element should be presented.

For HTML output, the content of the paragraph element shall be enclosed
in `p` tags if all the following conditions are satisfied:

 1. The _paragraph text_ does not contain any tag (open or close or
    self-closing tag) of any of the following HTML elements: `address`,
    `article`, `aside`, `blockquote`, `div`, `dl`, `fieldset`, `form`,
    `hr`, `nav`, ` noscript`, `ol`, `pre`, `section`, `table`, `ul`.

 2. There is no unmatched HTML tag (i.e. open tag without a close tag,
    or a close tag without an open tag) in the _paragraph text_

 3. At least one of the following conditions is satisfied:

     1. The first _non-space_ character of the _paragraph text_ is not
        part of a HTML tag (open or close or self-closing tag)

        (or)

     2. The last _non-space_ character of the _paragraph text_ is not
        part of a HTML tag (open or close or self-closing tag)

 4. At least one of the following conditions is satisfied:

     1. The containing HTML element (i.e. the direct parent element
        under which this paragraph will be placed in the output HTML) is
        not an `li` element

        (or)

     2. The last line in the _block-element line sequence_ is a _blank
        line_

If any of the above 4 conditions is not satisfied, the HTML output of
the paragraph shall be the same as the content of the paragraph, without
wrapping it in `p` tags.

### reference-resolution block

The _block-element line sequence_ for a reference-resolution block shall
have a single _line_.

The reference-resolution block does not result in any output by itself.
It is used to resolve the URLs of reference-style links and images in
the rest of the document.

The single _line_ in the _block-element line sequence_ shall match one
of the following regular expression patterns:

 1. Just the URL:
    `/^ *\[([^\\\[\]]|\\.)*\] *: *([^ <>]+|<[^<>]*>) *$/`

    Examples:  
    `[ref id]: http://example.net/`  
    `[ref \[ id]: <http://example.net/>`  
    `[ref \] id]: mailto:someone@somewhere.net`  

 2. URL followed by text:
    `/^ *\[([^\\\[\]]|\\.)*\] *: *([^ <>]+|<[^<>]*>) +([^ ].*)$/`

    Examples:  
    `[ref id]: http://example.net/ "Title"`  
    `[ref \[ id]: http://example.net/ 'Title'`  
    `[ref \] id]: http://example.net/ "Title with \"escaped\" quotes"`  
    `[ref id]: http://example.net/ "Title" followed by random "(ignored)" text`  
    `[ref id]: http://example.net/ just random ignored text`  

In case of either pattern, the matching substring for the first
parenthesized subexpression in the pattern is _simplified_ to obtain the
_reference id string_. The matching substring for the second
parathesized subexpression is called the _unprocessed url string_.

Any `<`, `>` or _whitespace_ characters in the _unprocessed url string_
are removed, and the resultant string is called the _link url string_.

In case the match is with the second regular expression pattern, the
matching substring for the third parenthesized subexpression in the
pattern is called the _trailing string_. If the _trailing string_ begins
with a _quoted string_, the _enclosed string_ of the _quoted string_ is
called the _link title string_, and the rest of the _trailing string_ is
ignored. If the _trailing string_ does not begin with a _quoted string_,
the whole of the _trailing string_ is ignored, and the _link title
string_ is said to be _null_.

The _reference id string_ is said to be associated with the _link url
string_ and the _link title string_. A new entry is added to the _link
reference association map_ with the _reference id string_ as the key,
and the _link url string_ and the _link title string_ as values, unless
the _link reference association map_ already has an entry with the
_reference id string_ as the key.

The **link reference association map** is an associative array that
contains data from all the reference-resolution blocks in the document,
that helps in mapping a _reference id_ to the _link url_ and _link
title_ that the _reference id_ represents. It is used to resolve link
references elsewhere in the document (either in a _closing referential
link tag_, or in an _image tag_).

### null block

The  _block-element line sequence_ for a null block element shall have a
single _blank line_.

A null block does not result in any output.

## Identifying span-elements

A **text run** is a sequence of span-level vfmd constructs in a
paragraph block.

To interpret a sequence of characters, called the **input character
sequence**, as a _text run_, we need to identify the type and extent of
the span-elements in the input.

Some characters in the _input character sequence_ form the text content,
and the other characters denote how the text content is to be "marked
up". The characters that denote "mark up" form _span tags_, and the
other characters form _text fragments_.

There are three categories of _span tags_: _opening span tags_, _closing
span tags_ and _self-contained span tags_.

For example, consider the following _text run_:

    The **`ls` command** [_lists_ files](/ls-cmd).

The above _text run_ can be broken down into the following:

    "The "       : text fragment
    "**"         : opening span tag (emphasis)
    "`ls`"       : self-contained span tag (code)
    " command"   : text fragment
    "**"         : closing span tag (emphasis)
    "["          : opening span tag (link)
    "_"          : opening span tag (emphasis)
    "lists "     : text fragment
    "_"          : closing span tag (emphasis)
    "files"      : text fragment
    "](/ls-cmd)" : closing span tag (link)
    "."          : text fragment

To identify and interpret the _span tags_ in the _input character
sequence_, we make use of a **stack of potential opening span tags**.
Each node in the _stack of potential opening span tags_ eventually might
or might not get interpreted as an _opening span tag_. If a
corresponding _closing span tag_ is identified, the node gets
interpreted as an _opening span tag_; else, it gets identified as a
_text fragment_.

Initially, the _stack of potential opening span tags_ is empty. The
stack grows upwards: the bottommost node in the stack is the first one
added to the stack, and pushing a node onto the stack places it on top
of the stack. Each node in the stack contains the following fields:

 1. A _tag string_ that contains the substring of the _input character
    sequence_ that forms the _span tag_
 2. A _node type_ that indicates what types of span element this node
    might be opening, the possible values being: _asterisk emphasis
    node_, _underscore emphasis node_, _link node_ or _raw html node_
 3. A _linked content start position_ that is set only if the _node
    type_ is equal to _link node_

The topmost node in the stack is called the _top node_.

A node _n_ is said to be the _topmost node_ of type _t_ if, and only if,
all the following conditions are satisfied:

 1. The _node type_ of _n_ is equal to _t_
 2. The node _n_ is present in the _stack of potential opening span tags_
 3. There is no other node _m_ (where _n_ != _m_) such that both the
    following are true:
     1. The node _m_ is above the node _n_ in the _stack of potential
        opening span tags_
     2. The _node type_ of _m_ is equal to _t_

The _topmost node_ of type _t_ is said to be _null_ if, and only if, the
_stack of potential opening span tags_ does not contain any node whose
_node type_ is equal to _t_.

To identify and interpret the _span tags_ in the _input character
sequence_, we follow the procedure described in the next subsection,
_Identifying and interpreting span tags_.

### Identifying and interpreting _span tags_

In this section, we discuss the procedure to identify and interpret the
_span tags_ in the _input character sequence_.

The procedure involves iterating over the characters in the _input
character sequence_. The current character position in the _input
character sequence_ is called the _current-position_. A
_current-position_ value of 1 indicates that we are going to process the
first character in the _input character sequence_.  The substring of the
_input character sequence_ starting from and including the character at
the _current-position_ and ending at the end of the _input character
sequence_ is called the _remaining-character-sequence_.

To identify and interpret the _span tags_, the following procedure is
to be followed:

 1. Set _current-position_ as 1
 2. Set _is-potential-span-tag_ as false
 3. If the character at the _current-position_ is either an unescaped
    `[` (open square bracket) character, or an unescaped `]` (close
    square bracket) character, then set _is-potential-span-tag_ as true
    and follow the procedure discussed in _Handling potential link 
    tags_
 4. If the character at the _current-position_ is either an unescaped
    `*` (asterisk) character, or an unescaped `_` (underscore or low
    line) character, then set _is-potential-span-tag_ as true and follow
    the procedure discussed in _Handling potential emphasis tags_
 5. TODO: Code
 6. TODO: Image
 7. TODO: Automatic links (\<http://link\> or http\://link)
 8. TODO: Raw HTML
 9. If _is-potential-span-tag_ is true, the above steps (3-7) should
    have identified either a _span tag candidate_ or a _text fragment_;
    Increment the _current-position_ by the length of the _span tag
    candidate_ or the _text fragment_ that was identified
 10. If _is-potential-span-tag_ is false, the character at the _current
     position_ is identified as being part of a _text fragment_;
     Increment the _current-position_ by 1
 11. If the _current-position_ is greater than the length of the _input
     character sequence_, then stop this procedure; else, go to Step 2.

#### Handling potential link tags

In this section, we discuss how to identify _span tags_ related to
links. This section assumes that the character at the _current-position_
is either an unescaped `[` character or an unescaped `]` character.

If the character at the _current-position_ is a `[` character, it might
indicate  an _opening link tag_, as described below:

 1. The `[` character at the _current-position_ is identified as a
    _span tag candidate_, which can potentially get interpreted as
    an _opening link tag_ in the future

 2. A new node is pushed onto the _stack of potential opening span
    tags_ with the following properties:

     1. The _tag string_ of the node is set to the `[` character at
        the current position
     2. The _node type_ of the node is set as _link node_
     3. The _linked content start position_ of the node is set to
        ( _current-position_ + 1 )

If the character at the _current-position_ is a `]` character, it might
indicate the start of a _closing link tag_, as described below:

 1. If the _topmost node_ of type _link node_ is
    not _null_, and the if the _remaining-character-sequence_ matches
    any of the following regular expression patterns:

     1. With trailing empty square brackets:
        `/^(\]\s*\[\s*\])/`

        Example:  
        `][]`

     2. With no trailing opening bracket:
        `/^(\])\s*[^\[\(]/`

        Example:  
        `] a`
    
     3. At the very end:
        `/^(\]\s*)$/`

        Example:  
        `]`
     
    then the following is done:
        
    <!-- For some reason, Redcarpet requires a comment here to correctly
    display the following list -->

     1. The matching substring for the first and only parenthesized
        subexpression in the matching pattern is identified as a _span
        tag candidate_, and interpreted as a _closing span tag_, or more
        specifically, as a **closing link tag**

     2. If the _topmost node_ of type _link node_ is not already the
        _top node_, all nodes above it are popped off and interpreted as
        _text fragments_

     3. The _top node_ (which should have its _node type_ equal to _link
        node_) is interpreted as an _opening span tag_, or more
        specifically, as an **opening link tag**

     4. The _closing link tag_ is said to correspond to the _opening
        link tag_, and any _span tags_ or _text fragments_ occuring
        between the _opening link tag_ and the _closing link tag_ are
        considered to form the _enclosed content_ of the link

     5. Let _reference id start position_ be the _linked content start
        position_ of the _top node_; Let _reference id end position_ be
        ( _current-position_ - 1 ). The substring of the _input
        character sequence_ starting from the _reference id start
        position_ and ending at the _reference id end position_, both
        inclusive, is called the _reference id string_.
        
        The _reference id string_ shall be used to look up the actual
        link url and link title from the _link reference association
        map_.

        If the _link reference association map_ contains an entry for
        _reference id string_, then the output shall have the _enclosed
        content_ linked to the link url and link title specified in the
        entry for the _reference id string_ in the _link reference
        association map_.
        
        If the _link reference association map_ does not contain an
        entry for _reference id string_, then the output shall have the
        _enclosed content_ appear as text, without being linked,
        enclosed within the text forming the _opening link tag_ and the
        text forming the _closing link tag_.

     6. The _top node_ is popped off

 2. If the _topmost node_ of type _other link node_ is not _null_, and
    if the _remaining-character-sequence_ matches the regular expression
    pattern `/^\]\s*\[(([^\\\[\]]|\\.)*)\]/` (Example: `] [ref id]`),
    then the following is done:

     1. The matching substring for the whole of the pattern is
        identified as a _span tag candidate_, and interpreted as a
        _closing span tag_, or more specifically, as a **closing link
        tag**

     2. The matching substring for the first and only parenthesized
        subexpression in the pattern is _simplified_ to obtain the
        _reference id string_

     3. If the _topmost node_ of type _link node_ is not already
        the _top node_, all nodes above it are popped off and
        interpreted as _text fragments_

     4. The _top node_ (which should have its _node type_ equal to
        _link node_) is interpreted as an _opening span tag_, or
        more specifically, as an **opening link tag**

     5. The _closing link tag_ is said to correspond to the _opening
        link tag_, and any _span tags_ or _text fragments_ occuring
        between the _opening link tag_ and the _closing link tag_ are
        considered to form the _enclosed content_ of the link

     6. The _reference id string_ shall be used to look up the actual
        link url and link title from the _link reference association
        map_. 

        If the _link reference association map_ contains an entry for
        _reference id string_, then the output shall have the _enclosed
        content_ linked to the link url and link title specified in the
        entry for the _reference id string_ in the _link reference
        association map_.
        
        If the _link reference association map_ does not contain an
        entry for _reference id string_, then the output shall have the
        _enclosed content_ appear as text, without being linked,
        enclosed within the text forming the _opening link tag_ and the
        text forming the _closing link tag_.

     7. The _top node_ is popped off

 3. If the _topmost node_ of type _other link node_ is not _null_, and
    if both the following conditions are satisfied:

     1. The _remaining-character-sequence_ matches one of the following
        regular expression patterns:

         1. URL without angle brackets: `/^\]\s*\(\s*([^\(\)<>\s]+)([\)\s].*)$/`

            Example: `] (http://www.example.net` + _residual-link-attribute-sequence_

         2. URL within angle brackets: `/^\]\s*\(\s*<([^<>]*)>([\)].+)$/`
   
            Examples:  
            `](<http://example.net>` + _residual-link-attribute-sequence_  
            `] ( <http://example.net/?q=)>` + _residual-link-attribute-sequence_

        In case of either pattern, the matching substring for the first
        parenthesised subexpression shall be called the _unprocessed url
        string_, and the matching substring for the second parenthesized
        subexpression shall be called the
        _residual-link-attribute-sequence_.  Any _whitespace_ characters
        in the _unprocessed url string_ are removed, and the resultant
        string is called the _link url string_.

        The position at which the _residual-link-attribute-sequence_
        starts within the _remaining-character-sequence_ (i.e. the
        number of characters present in the
        _remaining-character-sequence_ before the start of the
        _residual-link-attribute-sequence_) shall be called the
        _url-pattern-match-length_.

     2. The _residual-link-attribute-sequence_ matches one of the
        following regular expression patterns:

         1. Just the closing paranthesis: `/^\s*\)/`

            Example: `)`

            If this is the matching pattern, the _title string_ is said
            to be _null_.
        
         2. Title and/or appended ignorable text and closing paranthesis: 
            `/^\s*((("(([^"\\]|\\.)*)")|('(([^'\\]|\\.)*)')|(([^"'\)\\]|\\.)*))+)\)/`

            Examples:  
            `"Title")`  
            `'Title')`  
            `"A (nice) \"title\" for the link")`  
            `'Title' followed by random ignored text)`  
            `"Title" followed by random "(ignored)" text)`  
            `just random ignored text)`  
            `just ignored text with \(escaped parentheses\))`

            If this is the matching pattern, the matching substring for
            the first parenthesized subexpression in the pattern is
            _trimmed_ to give the _attributes-string_. If the
            _attributes-string_ begins with a _quoted string_, then the
            _enclosed string_ of the _quoted string_ is called the
            _title string_. If the _attributes-string_ does not begin
            with a _quoted string_, then the _title string_ is said to
            be _null_.

        The number of characters in the
        _residual-link-attribute-sequence_ that were consumed in
        matching the whole of the matching pattern is called the
        _attributes-pattern-match-length_.

    then, the following is done:

    <!-- For some reason, Redcarpet requires a comment here to correctly
    display the following list -->

     1. Let _close-link-tag-length_ be equal to the sum of the
        _url-pattern-match-length_ and the
        _attributes-pattern-match-length_. The first
        _close-link-tag-length_ characters of the
        _remaining-character-sequence_ are collectively identified as a
        _span tag candidate_, and interpreted as a _closing span tag_,
        or more specifically, as a **closing link tag**

     2. If the _topmost node_ of type _link node_ is not already the
        _top node_, all nodes above it are popped off and interpreted as
        _text fragments_

     3. The _top node_ (which should have its _node type_ equal to _link
        node_) is interpreted as an _opening span tag_, or more
        specifically, as an **opening link tag**

     4. The _closing link tag_ is said to correspond to the _opening
        link tag_, and any _span tags_ or _text fragments_ occuring
        between the _opening link tag_ and the _closing link tag_ are
        considered to form the _enclosed content_ of the link

     5. The _link url string_ shall be used as the link url for the
        link.  If the _title string_ is not _null_, the _title string_
        shall be used as the title for the link. The output shall have
        the _enclosed content_ linked to the link url and link title.

     6. The _top node_ is popped off
        
 4. If none of the above 3 conditions are satisfied, then the `]` at the
    _current-position_ is interpreted as a _text fragment_

Thus, using this procedure, the `[` or `]` at the _current-position_ is
identified to be the start of either a _span tag candidate_ or a _text
fragment_. In some cases, the _span tag candidate_ is also interpreted
as one of these _span tags_: _opening link tag_ or _closing link tag_.

#### Handling potential emphasis tags

In this section, we discuss how to identify _span tags_ related to
emphasis. This section assumes that the character at the
_current-position_ is either an unescaped `*` character or an unescaped
`_` character.

We define a **word separator** character to be a unicode code point
whose 'General\_Category' unicode property has one of the following
values:

 1. One of: Zs, Zl, Zp (i.e. a 'Separator')

    (or)

 2. One of: Pc, Pd, Ps, Pe, Pi, Pf, Po (i.e. a 'Punctuation')

    (or)

 3. One of: Cc, Cf
    
For example, the _space_ character, the _line break_ character, `.`,
 `,`, `(`, `)` are all _word separator_ characters.

Given that the character at the _current-position_ is either `*` or `_` ,
then the _remaining character sequence_ will definitely match one of the following
regular expression patterns:

 1. At the end of the _input character sequence_:
    `/^([\*_]+)$/`

 2. In the middle of the _input character sequence_:
    `/^([\*_]+)([^\*_])/`

In case of either pattern, the matching substring for the first
parenthesized subexpression is called the _emphasis indicator
string_.

If the match is with the second regular expression pattern given
above, and if the single character in the matching substring for the
second parenthesized subexpression in the pattern is not a _word
separator_ character, then the _emphasis indicator string_ is said
to be _left-flanking_, else, the _emphasis indicator string_ is said
to be not _left-flanking_.

If the _current position_ is greater than 1, and if the character at
the previous position (i.e. at _current position_ minus 1) is not a
_word separator_ character, then the _emphasis indicator string_ is
said to be _right-flanking_, else, the _emphasis indicator string_
is said to be not _right-flanking_.

For example, the string `I'm ***Bond***, *** James***Bond` contains 4
_emphasis indicator strings_, each consisting of 3 `*` characters.  The
first _emphasis indicator string_ is _left-flanking_ (but not
_right-flanking_), the second _emphasis indicator string_ is
_right-flanking_ (but not _left-flanking_), the third _emphasis
indicator string_ is neither _left-flanking_ nor _right-flanking_, and
the fourth _emphasis indicator string_ is both _left-flanking_ and
_right-flanking_.

An _emphasis indicator string_ can contain both `*` and `_`
characters. When we split the _emphasis indicator string_ into
substrings composed of the same character, with no adjacent
substring having a common character, we get a list of _emphasis
tag strings_.

For example:

<table>
<tr>
    <th><em>emphasis indicator string</em></th>
    <th>List of <em>emphasis tag strings</em></th>
</tr>
<tr>
    <td><code>***</code></td>
    <td>[<code>***</code>]</td>
</tr>
<tr>
    <td><code>***___**</code></td>
    <td>[<code>***</code>, <code>___</code>, <code>**</code>]</td>
</tr>
<tr>
    <td><code>__*_**__</code></td>
    <td>[<code>__</code>, <code>*</code>, <code>_</code>, <code>**</code>, <code>__</code>]</td>
</tr>
</table>

An _emphasis tag string_ shall either be composed entirely of `*`
characters, or be composed entirely of `_` characters. If it's
composed entirely of `*` characters, the _constituent character_ of
the _emphasis tag string_ is said to be `*`. If it's composed
entirely of `_` characters, the _constituent character_ of the
_emphasis tag string_ is said to be `_`.

If the _emphasis indicator string_ is neither _left-flanking_ nor
_right-flanking_, the the _emphasis indicator string_ is interpreted as
a _text fragment_.

Similarly, if the _emphasis indicator string_ is both _left-flanking_
and _right-flanking_, the the _emphasis indicator string_ is interpreted
as a _text fragment_.

If the _emphasis indicator string_ is _left-flanking_ and not
_right-flanking_, then the _emphasis tag strings_ in the _emphasis
indicator string_ can potentially become _opening emphasis tags_. In
this case, the following shall be done:

 1. The _emphasis indicator string_ is identified as a _span tag
    candidate_, which can potentially get interpreted as one or more
    _opening emphasis tags_ in the future

 2. For each _emphasis tag string_ in the _emphasis indicator string_
    (listed in the order in which the _emphasis tag string_ appears in
    the _emphasis indicator string_) a new node is pushed onto the
    _stack of potential opening span tags_ with the following
    properties:

     1. The _tag string_ of the node is set to the _emphasis tag string_
     2. If the  _constituent character_ of the _emphasis tag string_ is
        `*`, then the _node type_ of the node is set as _asterisk
        emphasis node_; if the  _constituent character_ of the _emphasis
        tag string_ is `_`, then the _node type_ of the node is set as
        _underscore emphasis node_

If the _emphasis indicator string_ is _right-flanking_ and not
_left-flanking_, then the _emphasis tag strings_ in the _emphasis
indicator string_ can potentially be interpreted as _closing emphasis
tags_.  In this case, the following shall be done:

 1. Set _current-tag-string_ to the first _emphasis tag string_ in
    the _emphasis indicator string_

 2. If the _constituent character_ of the _current-tag-string_ is `*`,
    then the _matching emphasis node_ is the _topmost node_ of type
    _asterisk emphasis node_; if the _constituent character_ of the
    _current-tag-string_ is `_`, then the _matching emphasis node_ is
    the _topmost node_ of type _underscore emphasis node_

 3. If _matching emphasis node_ is _null_, then the _emphasis tag
    string_ is interpreted as a _text fragment_

 4. If the _matching emphasis node_ is not _null_, and if it is not
    already the _top node_, then all nodes above it are popped off and
    interpreted as _text fragments_
 
 5. If the _matching emphasis node_ is not _null_, the procedure
    described in _Matching opening and closing emphasis_ is followed.
    The _current-tag-string_ and/or the _top node_ can get modified in
    this process.

 6. If the _current-tag-string_ is empty, and if there are any more
    unprocessed _emphasis tag strings_ in the _emphasis indicator
    string_, set the _current-tag-string_ to the next
    _emphasis-tag-string_ 
 
 7. If the _current-tag-string_ is not empty, go to Step 2


##### Matching opening and closing emphasis

This section describes how the _current-tag-string_ is to be matched
with the _matching emphasis node_ at the top of the _stack of potential
opening span tags_.

In this section, the _top node_ is assumed to be of a _node type_ that
matches the _constituent character_ of the _current-tag-string_. If the
_constituent character_ of the _current-tag-string_ is `*`, the _node
type_ of the _top node_ should be _asterisk emphasis node_. If the
_constituent character_ of the _current-tag-string_ is `_`, the _node
type_ of the _top node_ should be _underscore emphasis node_.

Let the _tag string_ of the _top node_ be called the _top node tag
string_. The _top node tag string_ is compared with the
_current-tag-string_.

 1. If the _top node tag string_ and the _current-tag-string_ are
    exactly the same strings, then:

     1. The _top node_ is interpreted as an _opening span tag_, or more
        specifically, as an **opening emphasis tag**

     2. The _current-tag-string_ is interpreted as a _closing span tag_,
        or more specifically, as a **closing emphasis tag**
    
     3. The _closing emphasis tag_ is said to correspond to the _opening
        emphasis tag_, and any _span tags_ or _text fragments_ occuring
        between the _opening emphasis tag_ and the _closing emphasis
        tag_ are considered to constitute the emphasized content

     4. Set the _current-tag-string_ to _null_

     5. The _top node_ is popped off
        
 2. If the _top node tag string_ and the _current-tag-string_ are not
    exactly the same strings, and if the _current-tag-string_ is a
    substring of the _top node tag string_, then:
            
     1. Let the length of the _current-tag-string_ be called the
        _current-tag-string-length_.
        
        The last _current-tag-string-length_ characters of the _top node
        tag string_ are interpreted as an _opening span tag_, or more
        specifically, as an **opening emphasis tag**.

     2. The whole of the _current-tag-string_ is interpreted as a
        _closing span tag_, or more specifically, as a **closing
        emphasis tag**

     3. The _closing emphasis tag_ is said to correspond to the _opening
        emphasis tag_, and any _span tags_ or _text fragments_ occuring
        between the _opening emphasis tag_ and the _closing emphasis
        tag_ are considered to constitute the emphasized content

     4. Set the _current-tag-string_ to _null_

     5. The characters of the _top node tag string_ that were
        interpreted as the _opening emphasis tag_ are removed from the
        _tag string_ of the _top node_ while the _top node_ is still
        retained in the _stack of potential opening span tags_

 3. If the _top node tag string_ and the _current-tag-string_ are
    not exactly the same strings, and if the _top node tag string_
    is a substring of the _current-tag-string_, then:
    
     1. The _top node_ is interpreted as an _opening span tag_, or more
        specifically, as an **opening emphasis tag**

     2. Let the length of the _top node tag string_ be called the
        _top-node-tag-string-length_.

        The first _top-node-tag-string-length_ characters of the
        _current-tag-string_ interpreted as a _closing span tag_, or
        more specifically, as a **closing emphasis tag**.

     3. The _closing emphasis tag_ is said to correspond to the _opening
        emphasis tag_, and any _span tags_ or _text fragments_ occuring
        between the _opening emphasis tag_ and the _closing emphasis
        tag_ are considered to constitute the emphasized content

     4. The characters of the _current-tag-string_ that were interpreted as
        the _closing emphasis tag_ are removed from the  _current-tag-string_ 
        
     5. The _top node_ is popped off

 4. If any of the above conditions are satisfied, then we would have
    identified an _opening emphasis tag_ and a _closing emphasis tag_.
    The length of the _opening emphasis tag_ and the _closing emphasis
    tag_ should be the same. Let the length of the _opening emphasis
    tag_ or the _closing emphasis tag_ be called the
    _emphasis-tag-length_.

    The kind of emphasis that is to be applied is determined as follows:

     1. If the _emphasis-tag-length_ is 1, use an emphasis of kind
        _emphatic stress_. In HTML output, this shall be output as an
        `em` element.

     2. If the _emphasis-tag-length_ is 2, use an emphasis of kind
        _strong importance_. In HTML output, this shall be output as a
        `strong` element.

     3. If the _emphasis-tag-length_ is 3 or above, use both an emphasis
        of kind _strong importance_ and an emphasis of kind _emphatic
        stress_.  In HTML output, this shall be output as an `em`
        element nested within a `strong` element.


