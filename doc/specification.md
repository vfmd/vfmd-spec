<h1 id="vfmd-specification">vfmd specification</h1>

This spec describes the [vfmd Markdown syntax] in formal terms, and
is intended to be read by someone implementing this spec to parse or
otherwise programmatically interpret vfmd input. If you only intend
to write a document using vfmd, see the [userguide].

[vfmd Markdown syntax]: introduction.md
[userguide]: userguide.md

This document is organized as follows:

  * [Definitions]
  * [Identifying block-elements]
    * [The block-element line sequence]
    * [Type and extent of a block-element]
  * [Interpreting block-elements]
    * [atx-style header]
    * [setext-style header]
    * [code block]
    * [blockquote]
    * [horizontal rule]
    * [unordered list]
    * [ordered list]
    * [paragraph]
    * [reference-resolution block]
    * [null block]
  * [Identifying span-elements]
    * [Identifying and interpreting span tags]
      * [Handling potential link tags]
      * [Handling potential emphasis tags]
      * [Handling potential code-span tags]
      * [Handling potential image tags]
      * [Handling HTML tags]

<h2 id="definitions">Definitions</h2>

[Definitions]: #definitions

<h3 id="document">Document</h3>

[document]: #document

The input Markdown text is called the **document**.

The [document] contains Unicode text in UTF-8 encoding without any
leading Byte-Order-Mark. Any byte sequences in the input that are
invalid in UTF-8 encoding are filtered off and are ignored. Therefore,
for the following discussion, the [document] is considered to not have
any invalid byte sequences.

<h3 id="characters">Characters</h3>

[character]: #characters
[characters]: #characters

A **character** is an atomic unit of text specified as a Unicode code
point and encoded in UTF-8 encoding.

The [document] consists of a sequence of [characters], where the
[characters] may represent either markup or character data.

A `#x09 (TAB)` character in the input shall be treated as four
consecutive `#x20 (SPACE)` characters.

<span id="space">
A `#x20 (SPACE)` character is henceforth called a **space** character.
</span>
<span id="non-space">
Any character that is not a `#x20 (SPACE)` character is henceforth
called a **non-space** character.
</span>

The character sequence `#x0D #x0A (CRLF)` in the input shall be treated
as a single `#x0A (LF)` character.

<span id="line-break">
A `#x0A (LF)` character is called a **line break** character.
</span>
<span id="non-line-break">
Any [character] that is not a _line break_ character is called a
**non-line-break** character.
</span>

<span id="whitespace">
A **whitespace** character is one of the following characters: `#x09
(TAB)`, `#x0A (LF)`, `#x0C (FF)`, `#x0D (CR)` or `#x20 (SPACE)`.
</span>

<span id="string">
A **string** is a sequence of zero or more characters.
</span>

<span id="trimming">
**Trimming** a [string] means removing any leading or trailing
[whitespace] characters from the [string].
</span>
For example, trimming 
<code style="white-space: pre;">   yellow  </code> yields `yellow`;
trimming <code style="white-space: pre;">green  </code>
yields `green`. Trimming a [string] that does  not have any leading
or trailing [whitespace] has no effect on the [string]. Trimming a
[string] that is entirely composed of [whitespace] characters yields
an empty (zero-length) [string].

<span id="simplifying">
**Simplifying** a [string] means [trimming] the [string], and in
addition, replacing each sequence of internal [whitespace] characters in
the [string] with a single [space] character.
</span>
For example, simplifying 
<code style="white-space: pre;">     Amazing   Maurice  </code>
yields `Amazing Maurice`; simplifying 
<code style="white-space: pre;">educated   rodents   </code>
yields `educated rodents`.

<span id="escaping">**Escaping**</span> a [character] in a [string]
means placing a `\` (backslash) just before the [character] in the
[string], where the `\` used for escaping remains unescaped (i.e.
the escaping `\` shall not itself be preceded by an unescaped `\`).

<span id="quoted-string">A **quoted string**</span> is a [string] that
consists at least two characters and either

 1. begins with an [unescaped] `'` (single quote) character, and ends
    with an [unescaped] `'` character, and does not contain any other
    instance of an [unescaped] `'` character

    or

 2. begins with an [unescaped] `"` (double quote) character, and ends
    with an [unescaped] `"` character, and does not contain any other
    instance of an [unescaped] `"` character

<span id="enclosed-string">The substring of the [quoted string] that
excludes the first and the last character of the [quoted string] is
called the **enclosed string** of the [quoted string].</span>

[space]: #space
[non-space]: #non-space
[line break]: #line-break
[line breaks]: #line-break
[non-line-break]: #non-line-break
[whitespace]: #whitespace
[string]: #string
[trimming]: #trimming
[trimmed]: #trimming
[simplifying]: #simplifying
[simplified]: #simplifying
[escaping]: #escaping
[unescaped]: #escaping
[quoted string]: #quoted-string
[enclosed string]: #enclosed-string

<h3 id="lines">Lines</h3>

[line]: #lines
[lines]: #lines

A **line** is a sequence of [non-line-break] characters.

When we split the [document] on [line breaks], we get _lines_. The
[document] can then be seen as a sequence of _lines_, separated by [line
breaks].

<span id="blank-line">
If a [line] contains no [characters], or if all [characters] in the
[line] are [space] characters, that line is called a **blank line**.
</span>

[blank line]: #blank-line
[blank lines]: #blank-line

<h3 id="block-elements">Block-elements</h3>

[block-elements]: #block-elements

The [document] can be seen as a sequence of block-elements, where each
block-element consists of a sequence of one or more [lines].

There are different types of block-elements: 

- Paragraph
- Header (atx-style, or setext-style)
- Blockquote
- List
- Code block
- Horizontal rule
- Null block

To [identify the block-elements] in the [document], the [document] is seen
as a sequence of [lines].

[identify the block-elements]: #identifying-block-elements


<h2 id="identifying-block-elements">Identifying block-elements</h2>

[Identifying block-elements]: #identifying-block-elements

Given a sequence of [lines], henceforth
called the **input line sequence**, we need to identify the
block-elements in the input, identify the type of the block-elements and
identify which sub-sequence of lines in the input correspond to which
block-element.

[input line sequence]: #identifying-block-elements

<h3 id="block-element-line-sequence">The block-element line sequence</h3>

[The block-element line sequence]: #block-element-line-sequence
[block-element line sequence]: #block-element-line-sequence
[block-element line sequences]: #block-element-line-sequence

The [line] at which a block-element begins is called a **block-element
start line**. The [line] at which a block-element ends is called a
**block-element end line**. The sub-sequence of [lines] starting from
the _block-element start line_ and ending at the _block-element end
line_, both inclusive, constitute the **block-element line sequence**.

Every _block-element end line_ in the [input line sequence] is
immediately followed by a _block-element start line_, unless the
_block-element end line_ is the last line in the [input line sequence].
The first line in the [input line sequence] is a _block-element start
line_, and the last line in the [input line sequence] is a
_block-element end line_.

The idea is to break the [input line sequence] into a series of
_block-element line sequences_. Each _block-element line sequence_ shall
consist of only the [lines] that correspond to a particular
block-element.

[block-element start line]: #block-element-line-sequence
[block-element end line]: #block-element-line-sequence

<h3 id="type-and-extent-block-element">Type and extent of a block-element</h3>

[Type and extent of a block-element]: #type-and-extent-block-element

The type of the block-element is determined based on the [block-element
start line] and, in some cases, the line following the [block-element
start line].  The line at which the block should end (i.e.  the
corresponding [block-element end line]) is determined based on the
[block-element start line] and subsequent lines.

**Definitions:**
<span id="unordered-list-starter-pattern">
The regular expression pattern `/^( *[\*\-\+] +)[^ ]/` is called the
**unordered list starter pattern**.
</span>
<span id="ordered-list-starter-pattern">
The regular expression pattern `/^( *([0-9]+)\. +)[^ ]/` is called the
**ordered list starter pattern**.
</span>

[unordered list starter pattern]: #unordered-list-starter-pattern
[ordered list starter pattern]: #ordered-list-starter-pattern

The following rules are to be followed in determining the type of the
block-element and the [block-element end line]:

 1. If the [block-element start line] is a [blank line], then the body
    element is of type [**null block**]. The same line is the
    [block-element end line].
 
 2. If the [block-element start line] does not begin with four or more
    consecutive [space] characters, and if the [block-element start
    line] matches the regular expression pattern
    `/^ *\[([^\\\[\]]|\\.)*\] *: *([^ <>]+|<[^<>]*>)/`, then the
    block-element is of type [**reference-resolution block**]. The same
    line is the [block-element end line].

 3. If none of the above conditions apply, and if the [block-element
    start line] is not the last line in the [input line sequence], and
    is immediately followed by a succeeding line that matches the
    regular expression pattern `/^(-+|=+) *$/`, then the block-element
    is said to be of type [**setext-style header**], and the succeeding
    line is said to be the [block-element end line].

 4. If none of the above conditions apply, and if the [block-element
    start line] begins with four or more consecutive [space] characters,
    it signifies the start of a block-element of type [**code block**].
    The [block-element end line] is the next subsequent line in the
    [input line sequence], starting from and inclusive of the
    [block-element start line], that is immediately succeeded by a
    succeeding line that satisfies one of the following conditions:

     1. The succeeding line is not a [blank line], and it does not
        begin with four or more consecutive [space] characters
        
        (or)

     2. The succeeding line is a [blank line], and is immediately
        followed by a line that does not begin with four or more
        consecutive [space] characters

    If no such [block-element end line] is found, the last line in the
    [input line sequence] is the [block-element end line].

 5. If none of the above conditions apply, and if the first character
    of the [block-element start line] is a `#` character, it signifies
    the start of a block-element of type [**atx-style header**]. The
    same line is the [block-element end line].

 6. If none of the above conditions apply, and if the first
    [non-space] character in the [block-element start line] is a `>`
    character, then the block-element is of type [**blockquote**]. The
    [block-element end line] is the next subsequent line in the [input
    line sequence] that is a [blank line] and is immediately succeeded
    by a succeeding line that satisfies one of the following conditions:

     1. The succeeding line is a [blank line]
     
        (or)

     2. The succeeding line begins with four or more consecutive [space]
        characters
        
        (or)

     3. The first [non-space] character in the succeeding line is not
        a `>` character
    
    If no such [block-element end line] is found, the last line in the
    [input line sequence] is the [block-element end line].

 7. If none of the above conditions apply, and if the [block-element
    start line] contains three or more `*` characters, and is composed
    entirely of instances of the `*` character and optional [space]
    characters, then the line forms a block-element of type
    [**horizontal rule**].

    Similarly, if the [block-element start line] contains either three
    or more `-` characters, or three or more `_` characters, and is
    composed entirely of the same character and optional [space]
    characters, then the block-element is of type [horizontal rule].

    The [block-element end line] is the same as the [block-element start
    line].

 8. If none of the conditions specified above apply, and if the
    [block-element start line] matches the [unordered list starter
    pattern] \(i.e. the regular expression `/^( *[\*\-\+] +)[^ ]/`\)
    then the block-element is of type [**unordered list**]. The matching
    substring for the first and only parenthesized subexpression in the
    pattern is called the _unordered list starter string_. The number of
    [characters] in the _unordered list starter string_ is called the
    _unordered-list-starter-string-length_.

    For example, consider the following [block-element start line]
    \(which has three [space] characters, followed by an asterisk,
    followed by two [space] characters, followed by the word
    "Peanuts"\):
  
           *  Peanuts

    The _unordered list starter string_ in the above example consists of
    the first 6 characters of the line, i.e. the entire part before the
    word "Peanuts". The _unordered-list-starter-string-length_ is 6.
   
    The [block-element end line] is the next subsequent line in the
    [input line sequence], starting from and inclusive of the
    [block-element start line], that satisfies one of the following
    conditions:

     1. The line is a [blank line] and is immediately succeeded by
        another [blank line]

        (or)

     2. The line is a [blank line] and is immediately succeeded by a
        succeeding line that satisfies all of the following conditions:

         1. The succeeding line does not start with the _unordered list
            starter string_ seen in the [block-element start line]
         2. The first _unordered-list-starter-string-length_ characters
            of the succeeding line include [non-space] characters

        (or)

     3. The line is not a [blank line] and is immediately succeeded by
        a succeeding line that statisfies all of the following
        conditions:

         1. The succeeding line does not start with the _unordered list
            starter string_ seen in the [block-element start line]
         2. The first _unordered-list-starter-string-length_ characters
            of the succeeding line include [non-space] characters
         3. The succeeding line matches either the [unordered list
            starter pattern] or the [ordered list starter pattern]

    If no such [block-element end line] is found, the last line in the
    [input line sequence] is the [block-element end line].

 9. If none of the conditions specified above apply, and if the
    [block-element start line] matches the [ordered list starter
    pattern] \(i.e. the regular expression `/^( *([0-9]+)\. +)[^ ]/`\)
    then the block-element is of type [**ordered list**]. The length of
    the matching substring for the first (i.e. outer) parenthesized
    subexpression in the pattern is called the
    _ordered-list-starter-string-length_.
    
    For example, consider the following [block-element start line]
    \(which has three [space] characters, followed by the number '1',
    followed by a dot, followed by two [space] characters, followed by
    the word "Peanuts"\):
  
           1.  Peanuts

    The _ordered-list-starter-string-length_ in the above example is 7.
   
    The [block-element end line] is the next subsequent line in the
    [input line sequence], starting from and inclusive of the
    [block-element start line], that satisfies one of the following
    conditions:

     1. The line is a [blank line] and is immediately succeeded by
        another [blank line]

        (or)

     2. The line is a [blank line] and is immediately succeeded by a
        succeeding line that satisfies all of the following conditions:

         1. The succeeding line does not match the [ordered list starter
            pattern]
         2. The first _ordered-list-starter-string-length_ characters of
            the succeeding line include [non-space] characters

        (or)

     3. The line is not a [blank line] and is immediately succeeded by
        a succeeding line that satisfies all of the following
        conditions:

         1. The succeeding line does not match the [ordered list starter
            pattern]
         2. The first _ordered-list-starter-string-length_ characters of
            the succeeding line include [non-space] characters
         3. The succeeding line matches the [unordered list starter
            pattern]

    If no such [block-element end line] is found, the last line in the
    [input line sequence] is the [block-element end line].

10. If none of the above conditions apply, then the block-element is of
    type [**paragraph**].

    In order to find the [block-element end line], we need to make use
    of a HTML parser. To the HTML parser, we feed the characters of each
    line, starting from the [block-element start line]. After feeding
    all characters of every line, we feed a `#x0A (LF)` character (i.e.
    a [line break]) to the HTML parser, and observe the state of the
    HTML parser.

    Of the the many possible states of a HTML parser, the following is
    the list of states that we are interested in:

     1. Within a HTML tag (open / close / self-closing tag)
     2. Within the contents of a HTML element, but not within a HTML
        open or close tag
     3. Within a HTML comment
     4. Outside of any HTML element or comment
 
    The [block-element end line] is the next subsequent [line] in the
    [input line sequence], starting from and inclusive of the
    [block-element start line], that satisfies all the following
    conditions:

    <!-- For some reason, Redcarpet requires a comment here to
         correctly display the following list -->

     1. At the end of feeding the [line] and a [line break] to the HTML
        parser, all the following conditions are satisfied:

         1. The HTML parser state is not "within a HTML tag"

         2. The HTML parser state is not "within a HTML comment"

         3. If the HTML parser state is "within the contents of a HTML
            element", then the containing HTML element or any of its
            ancestor elements is not one of the following HTML elements:
            `pre`, `script`, `style`

     2. The [line] is a [blank line], or is immediately succeeded by a
        succeeding line that satisfies at least one of the following
        conditions:

         1. The leftmost [non-space] character in the succeeding line is
            a `>` character

            (or)

         2. The succeeding line contains three or more `*` characters,
            and is composed entirely of instances of the `*` character
            and optional [space] characters (or similarly with `-` or
            `[` characters)


Using the above rules, the [input line sequence] is broken down into a
series of [block-element line sequences], and the block-element type of
each [block-element line sequence] is identified.

<h2 id="interpreting-block-elements">Interpreting block-elements</h2>

[Interpreting block-elements]: #interpreting-block-elements

This section assumes that the [input line sequence] has been broken down
into a series of [block-element line sequences], and that the
block-element type of each [block-element line sequence] has been
identified. For a given block-element type, the procedure to interpret a
[block-element line sequence] is discussed in this section.

<h3 id="atx-style-header">atx-style header</h3>

[atx-style header]: #atx-style-header
[**atx-style header**]: #atx-style-header

The  [block-element line sequence] for an atx-style header shall have a
single line that starts with a `#` character.

The single [line] in the [block-element line sequence] shall match one
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
    shall be [trimmed] to give a _header text run_. The result of
    processing the _header text run_ as a [text run] shall form the
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

    For example, if the complete [line] reads:

        #####

    then the corresponding HTML output shall be:

        <h5></h5>

<h3 id="setext-style-header">setext-style header</h3>

[setext-style header]: #setext-style-header
[**setext-style header**]: #setext-style-header

The  [block-element line sequence] for a setext-style header shall have
exactly two lines, with the second line beginning with either a `-`
character or a `=` character.

The first line shall be [trimmed] to give a _header text run_. The
result of processing the _header text run_ as a [text run] shall form
the content of the header element.

If the second line starts with the `=` character, the heading level
shall be 1. If the second line starts with the `-` character, the
heading level shall be 2. No other heading levels are possible in a
setext-style header.

For example, consider the following pairs of [lines]:

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

<h3 id="code-block">code block</h3>

[code block]: #code-block
[**code block**]: #code-block

Each line in the  [block-element line sequence] for a code block element
shall either be a [blank line] or a [line] beginning with four or more
[space] characters.

For each line in the [block-element line sequence], the leading four
[space] characters are removed, if present, and the resulting sequence
of [lines], separated by [line breaks], forms the content of the code
block.

For example, if the following sequence of [lines] form the code block
element:

        int main() {
            return 42;
        }

The corresponding HTML output shall be:

    <pre><code>int main() {
        return 42;
    }
    </code></pre>

<h3 id="blockquote">blockquote</h3>

[blockquote]: #blockquote
[**blockquote**]: #blockquote

The [block-element line sequence] for a blockquote element shall
have one or more [lines], some of which might have the `>` character as
the first [non-space] character in the [line].

If the last [line] in the [block-element line sequence] is a [blank
line], the last [line] is ignored.

Each [line] in the [block-element line sequence] is processed to produce
a modified sequence of [lines], called the _blockquote-processed line
sequence_. The following processing is to be done for each [line]:

 1. If the [line] matches the regular expression `/^ *> /`, then the
part of the [line] that matches the said regular expression shall be removed
from the line  
 2. If the pattern in (1) above is not satisfied, and if the [line] matches
the regular expression `/^ *>/`, then the part of the [line] that
matches the said regular expression shall be removed from the line

The _blockquote-processed line sequence_ obtained this way can be
considered as the [input line sequence] for a sequence of block-elements
nested within the blockquote. The result of interpreting that [input
line sequence] further into block-elements shall form the content of the
blockquote element.

For example, consider the following [block-element line sequence]:

      > In Perl, a Hello World is 
      > written as follows:
      >
      >     print "Hello World!\n";

After processing each line in the above [block-element line sequence],
the _blockquote-processed line sequence_ obtained is as follows:

    In Perl, a Hello World is 
    written as follows:
    
        print "Hello World!\n";

When we treat the _blockquote-processed line sequence_ as an [input line
sequence], we can recognize nested block elements in it of type
paragraph and code block. The HTML equivalent for the [lines] in the
_blockquote-processed line sequence_ is as follows:

    <p>In Perl, a Hello World is 
    written as follows:</p>
    
    <pre><code>print "Hello World!\n";
    </code></pre>

Therefore, the HTML equivalent for the given  [block-element line
sequence] is:

    <blockquote>
    <p>In Perl, a Hello World is 
    written as follows:</p>
    
    <pre><code>print "Hello World!\n";
    </code></pre>
    </blockquote>

<h3 id="horizontal-rule">horizontal rule</h3>

[horizontal rule]: #horizontal-rule
[**horizontal rule**]: #horizontal-rule

The  [block-element line sequence] for a horizontal rule element shall
have a single [line] that is composed entirely of either `*`, `-` or `_`
characters, along with optional [space] characters.

The output shall consist of a horizontal rule.

For example:

    Last line of a paragraph.

    * * *

    First line of a paragraph.

The corresponding HTML output shall be:

    <p>Last line of a paragraph.</p>

    <hr/>

    <p>First line of a paragraph.</p>

<h3 id="unordered-list">unordered list</h3>

[unordered list]: #unordered-list
[**unordered list**]: #unordered-list

The [block-element line sequence] for an unordered list block shall have
one or more [lines].

The first line in the [block-element line sequence] would match the
[unordered list starter pattern] \(i.e. the regular expression
`/^( *[\*\-\+] +)[^ ]/`\). The matching substring for the first and only
parenthesized subexpression in that pattern is called the _unordered
list starter string_. The number of characters in the _unordered list
starter string_ is called the _unordered-list-starter-string-length_.

We first divide the [block-element line sequence] into a series of
_unordered list item line sequences_. The [lines] in a particular
_unordered list item line sequence_ correspond to one list item in the
list.

Every [line] in the [block-element line sequence] that starts with the
_unordered list starter string_ is called an _unordered list item start
line_. Each _unordered list item start line_ signifies the beginning of
a new _unordered list item line sequence_. An _unordered list item line
sequence_ consists of the sequence of [lines] starting from (and
inclusive of) an _unordered list item start line_, and ending at (and
excluding) the next subsequent _unordered list item start line_. If
there is no subsequent _unordered list item start line_, the _unordered
list item line sequence_ ends at the last [line] of the [block-element
line sequence].

We have now divided the [block-element line sequence] into a series of
_unordered list item line sequences_. The first [line] of each
_unordered list item line sequence_ starts with the _unordered list
starter string_.

Each [line] in the _unordered list item line sequence_ is processed to
produce a modified sequence of [lines], called the
_unordered-list-item-processed line sequence_. The following processing
is to be done for each [line]:

 1. If the [line] is the first line of the _unordered list item line
    sequence_:
    
    The [line] would start with the _unordered list starter string_.
    The _unordered list starter string_ shall be removed from the
    beginning of the [line].

 2. If the [line] is not the first line of the _unordered list item line
    sequence_:
    
    The [line] would start with zero or more [space] characters. The
    leading [space] characters, if any, should be removed as given
    below:

     1. If the number of leading [space] characters exceeds the
        _unordered-list-starter-string-length_, the number of leading
        [space] characters removed should be equal to the
        _unordered-list-starter-string-length_.
     2. If the number of leading [space] characters is less than or
        equal to the _unordered-list-starter-string-length_, all the
        leading [space] characters should be removed.

The _unordered-list-item-processed line sequence_ obtained this way can
be considered as the [input line sequence] for a sequence of
block-elements nested within the list item. The result of interpreting
that [input line sequence] further into block-elements shall form the
content of the list element. 

The list elements so obtained are combined into a sequence to form the
complete unordered list in the output.

For example, consider the following [block-element line sequence]:

    * First item 1

    * Second item 1
    Second item 2

          Code block

    * Third item 1

        * Nested item 1

The _unordered list starter string_ for the above example is an asterisk
followed by a single [space] character. The
_unordered-list-starter-string-length_ is 2.

The 1<sup>st</sup>, 3<sup>rd</sup> and 8<sup>th</sup> lines in the
[block-element line sequence] start with the _unordered list starter
string_, and are therefore _unordered list item start lines_. (The
10<sup>th</sup> line does contain the _unordered list starter string_,
but does not start with the _unordered list starter string_, so it's not
an _unordered list item start line_.)  Therefore, there are three
_unordered list item line sequences_ in the above example, as follows:

 1. The lines 1 and 2 form the first _unordered list item
    line sequence_
 2. The lines 3-7 form the second _unordered list item line sequence_
 3. The lines 8-10 form the third and last _unordered list item
    line sequence_

Each _unordered list item line sequence_ is then processed to obtain the
_unordered-list-item-processed line sequence_.  When we treat each
_unordered-list-item-processed line sequence_ as an [input line
sequence], we can recognize nested block elements in it.

The first _unordered list item line sequence_ looks like:

<pre><code>* First item 1

</code></pre>

To obtain the corresponding _unordered-list-item-processed line
sequence_, we need to remove the _unordered list starter string_ from
the beginning of the first line. Since the second line is a [blank
line], no processing is done on the second line.

The first _unordered-list-item-processed line sequence_ is therefore:

<pre><code>First item 1

</code></pre>

When this _unordered-list-item-processed line sequence_ is processed as
an [input line sequence] to identify block-elements in it, we get a
single paragraph block-element.  The corresponding HTML would be:

    <p>First item 1</p>

The second _unordered list item line sequence_ looks like:

<pre><code>* Second item 1
Second item 2

      Code block

</code></pre>

To obtain the corresponding _unordered-list-item-processed line
sequence_, we need to remove the _unordered list starter string_ from
the beginning of the first line, and remove leading [space] characters,
subject to a maximum of 2 [space] characters (because the
_unordered-list-starter-string-length_ is 2), from the subsequent lines.

The second _unordered-list-item-processed line sequence_ is therefore:

<pre><code>Second item 1
Second item 2

    Code block

</code></pre>

When this _unordered-list-item-processed line sequence_ is processed as
an [input line sequence] to identify block-elements in it, we get a
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
the beginning of the first line, and remove leading [space] characters,
subject to a maximum of 2 [space] characters, from the subsequent lines.

The third _unordered-list-item-processed line sequence_ is therefore:

    Third item 1

      * Nested item 1

When this _unordered-list-item-processed line sequence_ is processed as
an [input line sequence] to identify block-elements in it, we get a
paragraph followed by an unordered list.  The corresponding HTML would
be:

    <p>Third item 1</p>

    <ul>
        <li>Nested item 1</li>
    </ul>

Putting the content of all the list items together, the HTML equivalent
for the complete [block-element line sequence] of the unordered list in
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

<h3 id="ordered-list">ordered list</h3>

[ordered list]: #ordered-list
[**ordered list**]: #ordered-list

The [block-element line sequence] for an ordered list block shall have
one or more [lines].

The first line in the [block-element line sequence] would match the
[ordered list starter pattern] \(i.e. the regular expression
`/^( *([0-9]+)\. +)[^ ]/`\). The length of the matching substring for the
first (i.e. outer) parenthesized subexpression in the pattern is called
the _ordered-list-starter-string-length_. The matching substring for the
second (i.e. inner) parenthesized subexpression in the pattern is called
the _ordered list starting number_.

We first divide the [block-element line sequence] into a series of
_ordered list item line sequences_. The lines in a particular
_ordered list item line sequence_ correspond to one list item in the
list.

Every [line] in the [block-element line sequence] that satisfies all the
following conditions is called an _ordered list item start line_:

 1. The [line] matches the [ordered list starter pattern]
 2. The first _ordered-list-starter-string-length_ characters of the
    [line] include [non-space] characters

Each _ordered list item start line_ signifies the beginning of a new
_ordered list item line sequence_. An _ordered list item line sequence_
consists of the sequence of [lines] starting from (and inclusive of) an
_ordered list item start line_, and ending at (and excluding) the next
subsequent _ordered list item start line_. If there is no subsequent
_ordered list item start line_, the _ordered list item line sequence_
ends at the last [line] of the [block-element line sequence].

We have now divided the [block-element line sequence] into a series of
_ordered list item line sequences_. The first line of each _ordered list
item line sequence_ matches the [ordered list starter pattern].

Each [line] in the _ordered list item line sequence_ is processed to
produce a modified sequence of [lines], called the
_ordered-list-item-processed line sequence_. The following processing is
to be done for each [line]:

 1. If the [line] is the first line of the _ordered list item line
    sequence_:
    
    The [line] would match the [ordered list starter pattern]. The
    matching substring for the first (i.e. outer) parenthesized
    subexpression in the pattern shall be removed from the beginning of
    the [line].
 2. If the [line] is not the first line of the _ordered list item line
    sequence_:
    
    The [line] would start with zero or more [space] characters. The
    leading [space] characters, if any, should be removed as given
    below:

     1. If the number of leading [space] characters exceeds the
        _ordered-list-starter-string-length_, the number of leading
        [space] characters removed should be equal to the
        _ordered-list-starter-string-length_.
     2. If the number of leading [space] characters is less than or
        equal to the _ordered-list-starter-string-length_, all the
        leading [space] characters should be removed.

The _ordered-list-item-processed line sequence_ obtained this way can be
considered as the [input line sequence] for a sequence of block-elements
nested within the list item. The result of interpreting that [input line
sequence] further into block-elements shall form the content of the list
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

For example, consider the following [block-element line sequence]:

    1. First item 1

    2. Second item 1
    Second item 2

          Code block

    3. Third item 1

        1. Nested item 1

When we match the first line against the [ordered list starter pattern],
the matching substring for the first (i.e. outer) parenthesized
subexpression is obtained as <code style="whitespace: pre;">1. </code>
(i.e. the number '1', followed by a dot, followed by a single [space]
character). The _ordered-list-starter-string-length_ is therefore 3.
Also, the _ordered list starting number_ is identified as the number
'1'.

The 1<sup>st</sup>, 3<sup>rd</sup>, 8<sup>th</sup> and 10<sup>th</sup>
lines in the [block-element line sequence] match the _ordered list
starter pattern_, but only the 1<sup>st</sup>, 3<sup>rd</sup> and
8<sup>th</sup> lines are such that the first 3 characters of the line
include [non-space] characters. So only the 1<sup>st</sup>,
3<sup>rd</sup> and 8<sup>th</sup> lines are are _ordered list item start
lines_.  Therefore, there are three _ordered list item line sequences_
in the above example, as follows:

 1. The lines 1 and 2 form the first _ordered list item line sequence_
 2. The lines 3-7 form the second _ordered list item line sequence_
 3. The lines 8-10 form the third and last _ordered list item line
    sequence_

Each _ordered list item line sequence_ is then processed to obtain the
_ordered-list-item-processed line sequence_.  When we treat each
_ordered-list-item-processed line sequence_ as an [input line sequence],
we can recognize nested block elements in it.

The first _ordered list item line sequence_ looks like:

<pre><code>1. First item 1

</code></pre>

To obtain the corresponding _ordered-list-item-processed line sequence_,
we need to match the first line against the [ordered list starter
pattern] and remove the matching substring for the first parenthesized
subexpression.  The matching substring in this case is <code
style="whitespace: pre;">1. </code> (i.e. the number '1', followed by a
dot, followed by a single space character). Since the second line is a
[blank line], no processing is done on the second line.

The first _ordered-list-item-processed line sequence_ is therefore:

<pre><code>First item 1

</code></pre>

When this _ordered-list-item-processed line sequence_ is processed as an
[input line sequence] to identify block-elements in it, we get a single
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
subexpression. The matching substring in this case is <code
style="whitespace: pre;">2. </code> (i.e. the number '2', followed by a
dot, followed by a single space character). From subsequent lines, we
need to remove leading [space] characters, subject to a maximum of 3
[space] characters (because the _ordered-list-starter-string-length_ is
3).

The second _ordered-list-item-processed line sequence_ is therefore:

<pre><code>Second item 1
Second item 2

    Code block

</code></pre>

When this _ordered-list-item-processed line sequence_ is processed as
an [input line sequence] to identify block-elements in it, we get a
paragraph followed by a code block.  The corresponding HTML would be:

    <p>Second item 1
    Second item 2</p>

    <pre><code>Code block
    </code></pre>

The third _ordered list item line sequence_ looks like:

    3. Third item 1

        1. Nested item 1

To obtain the corresponding _ordered-list-item-processed line sequence_,
we need to match the first line against the [ordered list starter
pattern] and remove the matching substring for the first parenthesized
subexpression. The matching substring in this case is <code
style="whitespace: pre;">3. </code> (i.e. the number '3', followed by a
dot, followed by a single space character). From subsequent lines, we
need to remove leading [space] characters, subject to a maximum of 3
[space] characters (because the _ordered-list-starter-string-length_ is
3).

The third _ordered-list-item-processed line sequence_ is therefore:

    Third item 1

     1. Nested item 1

When this _ordered-list-item-processed line sequence_ is processed as
an [input line sequence] to identify block-elements in it, we get a
paragraph followed by an ordered list.  The corresponding HTML would
be:

    <p>Third item 1</p>

    <ol>
        <li>Nested item 1</li>
    </ol>

Putting the content of all the list items together, the HTML equivalent
for the complete [block-element line sequence] of the ordered list in
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

<h3 id="paragraph">paragraph</h3>

[paragraph]: #paragraph
[**paragraph**]: #paragraph

The [block-element line sequence] for a paragraph block shall have
one or more [lines].

The lines in the [block-element line sequence] are joined together into
a single sequence of [characters], with a [line break] after each line.
The resulting sequence of [characters] is called the _paragraph text_.
The result of interpreting the _paragraph text_ as a [text run] shall
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

     1. The first [non-space] character of the _paragraph text_ is not
        part of a HTML tag (open or close or self-closing tag)

        (or)

     2. The last [non-space] character of the _paragraph text_ is not
        part of a HTML tag (open or close or self-closing tag)

 4. At least one of the following conditions is satisfied:

     1. The containing HTML element (i.e. the direct parent element
        under which this paragraph will be placed in the output HTML) is
        not an `li` element

        (or)

     2. The last line in the [block-element line sequence] is a [blank
        line]

If any of the above 4 conditions is not satisfied, the HTML output of
the paragraph shall be the same as the content of the paragraph, without
wrapping it in `p` tags.

<h3 id="reference-resolution-block">reference-resolution block</h3>

[reference-resolution block]: #reference-resolution-block
[**reference-resolution block**]: #reference-resolution-block

The [block-element line sequence] for a reference-resolution block shall
have a single [line].

The reference-resolution block does not result in any output by itself.
It is used to resolve the URLs of reference-style links and images in
the rest of the document.

The single [line] in the [block-element line sequence] shall match one
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
parenthesized subexpression in the pattern is [simplified] to obtain the
_reference id string_. The matching substring for the second
parathesized subexpression is called the _unprocessed url string_.

Any `<`, `>` or [whitespace] characters in the _unprocessed url string_
are removed, and the resultant string is called the _link url string_.

In case the match is with the second regular expression pattern, the
matching substring for the third parenthesized subexpression in the
pattern is called the _trailing string_. If the _trailing string_ begins
with a [quoted string], the [enclosed string] of the [quoted string] is
called the _link title string_, and the rest of the _trailing string_ is
ignored. If the _trailing string_ does not begin with a [quoted string],
the whole of the _trailing string_ is ignored, and the _link title
string_ is said to be _null_.

The _reference id string_ is said to be associated with the _link url
string_ and the _link title string_. A new entry is added to the [link
reference association map] with the _reference id string_ as the key,
and the _link url string_ and the _link title string_ as values, unless
the [link reference association map] already has an entry with the
_reference id string_ as the key.

The <span id="link-reference-association-map">**link reference
association map**</span> is an associative array that contains data from
all the reference-resolution blocks in the document, that helps in
mapping a _reference id_ to the _link url_ and _link title_ that the
_reference id_ represents. It is used to resolve link references
elsewhere in the document (either in a [closing link tag],
or in an [image tag]).

[link reference association map]: #link-reference-association-map

<h3 id="null-block">null block</h3>

[null block]: #null-block
[**null block**]: #null-block

The  [block-element line sequence] for a null block element shall have a
single [blank line].

A null block does not result in any output.

<h2 id="identifying-span-elements">Identifying span-elements</h2>

[Identifying span-elements]: #identifying-span-elements
[text run]: #identifying-span-elements
[input character sequence]: #identifying-span-elements

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

For example, consider the following _input character sequence_:

    The **`ls` command** [_lists_ files](/ls-cmd).

The above _input character sequence_ can be broken down into the
following:

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
sequence_, we make use of a 
<span id="stack-of-potential-opening-span-tags">
**stack of potential opening span tags**</span>.
Each node in the _stack of potential opening span tags_ eventually might
or might not get interpreted as an _opening span tag_. If a
corresponding _closing span tag_ is identified, the node gets
interpreted as an _opening span tag_; else, it gets identified as a
_text fragment_.

Initially, the _stack of potential opening span tags_ is empty. The
stack grows upwards: the bottommost node in the stack is the first one
added to the stack, and pushing a node onto the stack places it on top
of the stack. 

<span id="stack-node-properties">
Each node in the stack contains the following properties:</span>

 1. A _tag string_ that contains the substring of the _input character
    sequence_ that forms the _span tag_
 2. A _node type_ that indicates what types of span element this node
    might be opening, the possible values being: _asterisk emphasis
    node_, _underscore emphasis node_, _link node_ or _raw html node_
 3. A _linked content start position_ that is set only if the _node
    type_ is equal to _link node_
 4. A _html tag name_ that is set only if the _node type_ is equal to
    _raw html node_

<span id="top-node">The topmost node in the [stack of potential opening
span tags] is called the _top node_.</span>

<span id="topmost-node-of-type">A node _n_ is said to be the _topmost
node_ of type _t_ if, and only if, all the following conditions are
satisfied:</span>

 1. The _node type_ of _n_ is equal to _t_
 2. The node _n_ is present in the [stack of potential opening span
    tags]
 3. There is no other node _m_ (where _n_ != _m_) such that both the
    following are true:
     1. The node _m_ is above the node _n_ in the [stack of potential
        opening span tags]
     2. The _node type_ of _m_ is equal to _t_, or the _node type_ of
        _m_ is equal to _raw html node_

The _topmost node_ of type _t_ is said to be _null_ if, and only if, any
of the following conditions is true:

 1. The [stack of potential opening span tags] does not contain any node
    whose _node type_ is equal to _t_

    (or)
 
 2. All nodes whose _node type_ is equal to _t_ in the [stack of
    potential opening span tags] have a _html node_ above them (where
    _html node_ means a node whose _node type_ is _raw html node_)

To identify and interpret the _span tags_ in the [input character
sequence], we follow the procedure described in the next subsection,
[Identifying and interpreting _span tags_].

[stack of potential opening span tags]: #stack-of-potential-opening-span-tags
[stack-node-properties]: #stack-node-properties
[top node]: #top-node
[topmost node of type]: #topmost-node-of-type

<h3 id="identifying-and-interpreting-span-tags">
Identifying and interpreting <em>span tags</em></h3>

[Identifying and interpreting _span tags_]: #identifying-and-interpreting-span-tags
[Identifying and interpreting span tags]: #identifying-and-interpreting-span-tags

In this section, we discuss the procedure to identify and interpret the
_span tags_ in the [input character sequence].

The procedure involves iterating over the characters in the [input
character sequence]. <span id="current-position">The current character
position in the [input character sequence] is called the
**current-position**.</span> A _current-position_ value of 1 indicates
that we are going to process the first character in the [input character
sequence]. <span id="remaining-character-sequence">The substring of the
[input character sequence] starting from and including the character at
the [current-position] and ending at the end of the [input character
sequence] is called the **remaining-character-sequence**.</span>

[current-position]: #current-position
[remaining-character-sequence]: #remaining-character-sequence

To identify and interpret the _span tags_, the following procedure is
to be followed:

 1. Set [current-position] as 1
 2. Set _is-potential-span-tag_ as false
 3. If the character at the [current-position] is either an unescaped
    `[` (open square bracket) character, or an unescaped `]` (close
    square bracket) character, then set _is-potential-span-tag_ as true
    and follow the procedure discussed in [Handling potential link 
    tags]
 4. If the character at the [current-position] is either an unescaped
    `*` (asterisk) character, or an unescaped `_` (underscore or low
    line) character, then set _is-potential-span-tag_ as true and follow
    the procedure discussed in [Handling potential emphasis tags]
 5. If the character at the [current-position] is an unescaped `` ` ``
    (backtick) character, then set _is-potential-span-tag_ as true and
    follow the procedure discussed in [Handling potential code-span
    tags]
 6. If the character at the [current-position] is an unescaped `!`
    (exclamation mark) character, and if the
    [remaining-character-sequence] matches the regular expression
    pattern `/!\[/`, then set _is-potential-span-tag_ as true and follow
    the procedure discussed in [Handling potential image tags]
 7. TODO: Automatic links (\<http://link\> or http\://link)
 8. If the character at the [current-position] is an unescaped `<` (left
    angle bracket) character, and if the [remaining-character-sequence]
    matches neither the regular expression pattern
    `/<(https?|ftp):\/\/\S]/` nor the regular expression pattern
    `/<mailto:\S/`, then set _is-potential-span-tag_ as true and follow
    the procedure discussed in [Handling HTML tags]
 9. If _is-potential-span-tag_ is true, the above steps (3-7) should
    have identified either a _span tag candidate_ or a _text fragment_;
    Increment the [current-position] by the length of the _span tag
    candidate_ or the _text fragment_ that was identified
 10. If _is-potential-span-tag_ is false, the character at the _current
     position_ is identified as being part of a _text fragment_;
     Increment the [current-position] by 1
 11. If the [current-position] is greater than the length of the _input
     character sequence_, then stop this procedure; else, go to Step 2.

<h4 id="handling-potential-link-tags">Handling potential link tags</h4>

[Handling potential link tags]: #handling-potential-link-tags
[closing link tag]: #handling-potential-link-tags

In this section, we discuss how to identify _span tags_ related to
links. This section assumes that the character at the [current-position]
is either an unescaped `[` character or an unescaped `]` character.

If the character at the [current-position] is a `[` character, it might
indicate  an _opening link tag_, as described below:

 1. The `[` character at the [current-position] is identified as a
    _span tag candidate_, which can potentially get interpreted as
    an _opening link tag_ in the future

 2. A new node is pushed onto the [stack of potential opening span
    tags] with the following [properties][stack-node-properties]:

     1. The _tag string_ of the node is set to the `[` character at
        the current position
     2. The _node type_ of the node is set as _link node_
     3. The _linked content start position_ of the node is set to
        ( [current-position] + 1 )

If the character at the [current-position] is a `]` character, it might
indicate the start of a _closing link tag_, as described below:

 1. If the [topmost node of type] _link node_ is not _null_, and the if
    the [remaining-character-sequence] matches any of the following
    regular expression patterns:

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

     2. If the [topmost node of type] _link node_ is not already the
        [top node], all nodes above it are popped off and interpreted as
        _text fragments_

     3. The [top node] (which should have its _node type_ equal to _link
        node_) is interpreted as an _opening span tag_, or more
        specifically, as an **opening link tag**

     4. The _closing link tag_ is said to correspond to the _opening
        link tag_, and any _span tags_ or _text fragments_ occuring
        between the _opening link tag_ and the _closing link tag_ are
        considered to form the _enclosed content_ of the link

     5. Let _reference id start position_ be the _linked content start
        position_ of the _top node_; Let _reference id end position_ be
        ( [current-position] - 1 ). The substring of the _input
        character sequence_ starting from the _reference id start
        position_ and ending at the _reference id end position_, both
        inclusive, is [simplified] to obtain the _reference id string_.

        The _reference id string_ shall be used to look up the actual
        link url and link title from the [link reference association
        map].

        If the [link reference association map] contains an entry for
        _reference id string_, then the output shall have the _enclosed
        content_ linked to the link url and link title specified in the
        entry for the _reference id string_ in the [link reference
        association map]. For HTML output, the link title should be
        [attribute-value-escaped].
        
        If the [link reference association map] does not contain an
        entry for _reference id string_, then the output shall have the
        _enclosed content_ without being part of a link,
        enclosed within the text forming the _opening link tag_ and the
        text forming the _closing link tag_.

     6. The [top node] is popped off

     7. All nodes with _node type_ equal to _link node_ are removed from
        the [stack of potential opening span tags]

 2. If the [topmost node of type] _link node_ is not _null_, and
    if the [remaining-character-sequence] matches the regular expression
    pattern `/^\]\s*\[(([^\\\[\]]|\\.)*)\]/` (Example: `] [ref id]`),
    then the following is done:

     1. The matching substring for the whole of the pattern is
        identified as a _span tag candidate_, and interpreted as a
        _closing span tag_, or more specifically, as a **closing link
        tag**

     2. The matching substring for the first (i.e. outer) parenthesized
        subexpression in the pattern is [simplified] to obtain the
        _reference id string_

     3. If the [topmost node of type] _link node_ is not already
        the [top node], all nodes above it are popped off and
        interpreted as _text fragments_

     4. The [top node] (which should have its _node type_ equal to
        _link node_) is interpreted as an _opening span tag_, or
        more specifically, as an **opening link tag**

     5. The _closing link tag_ is said to correspond to the _opening
        link tag_, and any _span tags_ or _text fragments_ occuring
        between the _opening link tag_ and the _closing link tag_ are
        considered to form the _enclosed content_ of the link

     6. The _reference id string_ shall be used to look up the actual
        link url and link title from the [link reference association
        map]. 

        If the [link reference association map] contains an entry for
        _reference id string_, then the output shall have the _enclosed
        content_ linked to the link url and link title specified in the
        entry for the _reference id string_ in the [link reference
        association map]. For HTML output, the link title should be
        [attribute-value-escaped].
        
        If the [link reference association map] does not contain an
        entry for _reference id string_, then the output shall have the
        _enclosed content_ without being part of a link,
        enclosed within the text forming the _opening link tag_ and the
        text forming the _closing link tag_. For HTML output, the text
        forming the _closing link tag_ should be [html-escaped] before
        being output.

     7. The [top node] is popped off

     8. All nodes with _node type_ equal to _link node_ are removed from
        the [stack of potential opening span tags]

 3. If the _topmost node_ of type _other link node_ is not _null_, and
    if both the following conditions are satisfied:

     1. The [remaining-character-sequence] matches one of the following
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
        _residual-link-attribute-sequence_.  Any [whitespace] characters
        in the _unprocessed url string_ are removed, and the resultant
        string is called the _link url string_.

        The position at which the _residual-link-attribute-sequence_
        starts within the [remaining-character-sequence] \(i.e. the
        number of characters present in the
        [remaining-character-sequence] before the start of the
        _residual-link-attribute-sequence_\) shall be called the
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
            [trimmed] to give the _attributes-string_. If the
            _attributes-string_ begins with a [quoted string], then the
            [enclosed string] of the [quoted string] is called the
            _title string_. If the _attributes-string_ does not begin
            with a [quoted string], then the _title string_ is said to
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
        [remaining-character-sequence] are collectively identified as a
        _span tag candidate_, and interpreted as a _closing span tag_,
        or more specifically, as a **closing link tag**

     2. If the [topmost node of type] _link node_ is not already the
        [top node], all nodes above it are popped off and interpreted as
        _text fragments_

     3. The [top node] (which should have its _node type_ equal to _link
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
        For HTML output, the link title should be
        [attribute-value-escaped].

     6. The [top node] is popped off

     7. All nodes with _node type_ equal to _link node_ are removed from
        the [stack of potential opening span tags]

 4. If none of the above 3 conditions are satisfied, then the `]` at the
    [current-position] is interpreted as a _text fragment_

Thus, using this procedure, the `[` or `]` at the [current-position] is
identified to be the start of either a _span tag candidate_ or a _text
fragment_. In some cases, the _span tag candidate_ is also interpreted
as one of these _span tags_: _opening link tag_ or _closing link tag_.

<h4 id="handling-potential-emphasis-tags">Handling potential emphasis tags</h4>

[Handling potential emphasis tags]: #handling-potential-emphasis-tags

In this section, we discuss how to identify _span tags_ related to
emphasis. This section assumes that the character at the
[current-position] is either an unescaped `*` character or an unescaped
`_` character.

<span id="word-separator">We define a **word separator** [character] to
be a unicode code point whose 'General\_Category' unicode property has
one of the following values:</span>

 1. One of: Zs, Zl, Zp (i.e. a 'Separator')

    (or)

 2. One of: Pc, Pd, Ps, Pe, Pi, Pf, Po (i.e. a 'Punctuation')

    (or)

 3. One of: Cc, Cf
    
For example, the _space_ character, the _line break_ character, `.`,
 `,`, `(`, `)` are all _word separator_ characters.

Given that the character at the [current-position] is either `*` or `_` ,
then the [remaining-character-sequence] will definitely match one of the
following regular expression patterns:

 1. At the end of the [input character sequence]:
    `/^([\*_]+)$/`

 2. In the middle of the [input character sequence]:
    `/^([\*_]+)([^\*_])/`

<span id="emphasis-indicator-string">
In case of either pattern, the matching substring for the first
parenthesized subexpression is called the _emphasis indicator
string_.</span>

<span id="left-flanking">
If the match is with the second regular expression pattern given
above, and if the single character in the matching substring for the
second parenthesized subexpression in the pattern is not a [word
separator] character, then the _emphasis indicator string_ is said
to be _left-flanking_, else, the _emphasis indicator string_ is said
to be not _left-flanking_.</span>

<span id="right-flanking">
If the [current-position] is greater than 1, and if the character at
the previous position (i.e. at [current-position] minus 1) is not a
[word separator] character, then the _emphasis indicator string_ is
said to be _right-flanking_, else, the _emphasis indicator string_
is said to be not _right-flanking_.</span>

For example, the string `I'm ***Bond***, *** James***Bond` contains 4
_emphasis indicator strings_, each consisting of 3 `*` characters.  The
first _emphasis indicator string_ is _left-flanking_ (but not
_right-flanking_), the second _emphasis indicator string_ is
_right-flanking_ (but not _left-flanking_), the third _emphasis
indicator string_ is neither _left-flanking_ nor _right-flanking_, and
the fourth _emphasis indicator string_ is both _left-flanking_ and
_right-flanking_.

<span id="emphasis-tag-string">
An [emphasis indicator string] can contain both `*` and `_`
characters. When we split the [emphasis indicator string] into
substrings composed of the same character, with no adjacent
substring having a common character, we get a list of _emphasis
tag strings_.</span>

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

<span id="constituent-character">
An [emphasis tag string] shall either be composed entirely of `*`
characters, or be composed entirely of `_` characters. If it's
composed entirely of `*` characters, the _constituent character_ of
the [emphasis tag string] is said to be `*`. If it's composed
entirely of `_` characters, the _constituent character_ of the
[emphasis tag string] is said to be `_`.</span>

[word separator]: #word-separator
[emphasis indicator string]: #emphasis-indicator-string
[left-flanking]: #left-flanking
[right-flanking]: #right-flanking
[emphasis tag string]: #emphasis-tag-string
[emphasis tag strings]: #emphasis-tag-string
[constituent character]: #constituent-character

If the [emphasis indicator string] is neither [left-flanking] nor
[right-flanking], the the [emphasis indicator string] is interpreted as
a _text fragment_.

Similarly, if the [emphasis indicator string] is both [left-flanking]
and [right-flanking], the the [emphasis indicator string] is interpreted
as a _text fragment_.

If the [emphasis indicator string] is [left-flanking] and not
[right-flanking], then the [emphasis tag strings] in the [emphasis
indicator string] can potentially become _opening emphasis tags_. In
this case, the following shall be done:

 1. The [emphasis indicator string] is identified as a _span tag
    candidate_, which can potentially get interpreted as one or more
    _opening emphasis tags_ in the future

 2. For each [emphasis tag string] in the [emphasis indicator string]
    \(listed in the order in which the [emphasis tag string] appears in
    the [emphasis indicator string]\) a new node is pushed onto the
    [stack of potential opening span tags] with the following
    [properties][stack-node-properties]:

     1. The _tag string_ of the node is set to the _emphasis tag string_
     2. If the [constituent character] of the [emphasis tag string] is
        `*`, then the _node type_ of the node is set as _asterisk
        emphasis node_; if the  [constituent character] of the _emphasis
        tag string_ is `_`, then the _node type_ of the node is set as
        _underscore emphasis node_

If the [emphasis indicator string] is [right-flanking] and not
[left-flanking], then the [emphasis tag strings] in the [emphasis
indicator string] can potentially be interpreted as _closing emphasis
tags_.  In this case, the following shall be done:

 1. Set _current-tag-string_ to the first [emphasis tag string] in
    the [emphasis indicator string]

 2. If the [constituent character] of the _current-tag-string_ is `*`,
    then the _matching emphasis node_ is the [topmost node of type]
    _asterisk emphasis node_; if the [constituent character] of the
    _current-tag-string_ is `_`, then the _matching emphasis node_ is
    the [topmost node of type] _underscore emphasis node_

 3. If _matching emphasis node_ is _null_, then the _emphasis tag
    string_ is interpreted as a _text fragment_

 4. If the _matching emphasis node_ is not _null_, and if it is not
    already the [top node], then all nodes above it are popped off and
    interpreted as _text fragments_
 
 5. If the _matching emphasis node_ is not _null_, the procedure
    described in [Matching opening and closing emphasis] is followed.
    The _current-tag-string_ and/or the [top node] can get modified in
    this process.

 6. If the _current-tag-string_ is empty, and if there are any more
    unprocessed [emphasis tag strings] in the [emphasis indicator
    string], set the _current-tag-string_ to the next
    [emphasis tag string]
 
 7. If the _current-tag-string_ is not empty, go to Step 2

Thus, using this procedure, a sequence of one or more `*` or `_`
characters at the [current-position] is identified to be the start of
either a _span tag candidate_ or a _text fragment_.

<h5 id="matching-opening-and-closing-emphasis">
Matching opening and closing emphasis</h5>

[Matching opening and closing emphasis]: #matching-opening-and-closing-emphasis

This section describes how the _current-tag-string_ is to be matched
with the _matching emphasis node_ at the top of the [stack of potential
opening span tags].

In this section, the [top node] is assumed to be of a _node type_ that
matches the [constituent character] of the _current-tag-string_. If the
[constituent character] of the _current-tag-string_ is `*`, the _node
type_ of the [top node] should be _asterisk emphasis node_. If the
[constituent character] of the _current-tag-string_ is `_`, the _node
type_ of the [top node] should be _underscore emphasis node_.

Let the _tag string_ of the [top node] be called the _top node tag
string_. The _top node tag string_ is compared with the
_current-tag-string_.

 1. If the _top node tag string_ and the _current-tag-string_ are
    exactly the same strings, then:

     1. The [top node] is interpreted as an _opening span tag_, or more
        specifically, as an **opening emphasis tag**

     2. The _current-tag-string_ is interpreted as a _closing span tag_,
        or more specifically, as a **closing emphasis tag**
    
     3. The _closing emphasis tag_ is said to correspond to the _opening
        emphasis tag_, and any _span tags_ or _text fragments_ occuring
        between the _opening emphasis tag_ and the _closing emphasis
        tag_ are considered to constitute the emphasized content

     4. Set the _current-tag-string_ to _null_

     5. The [top node] is popped off
        
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
        _tag string_ of the [top node] while the [top node] is still
        retained in the [stack of potential opening span tags]

 3. If the _top node tag string_ and the _current-tag-string_ are
    not exactly the same strings, and if the _top node tag string_
    is a substring of the _current-tag-string_, then:
    
     1. The [top node] is interpreted as an _opening span tag_, or more
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
        
     5. The [top node] is popped off

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

<h4 id="handling-potential-code-span-tags">
Handling potential code-span tags</h4>

[Handling potential code-span tags]: #handling-potential-code-span-tags

In this section, we discuss how to identify _span tags_ related to
code-spans. This section assumes that the character at the
[current-position] is an unescaped `` ` `` character.

The following procedure is followed:

 1. The [remaining-character-sequence] shall match one of the following
    regular expression patterns:
    
     1. Backticks followed by a non-backtick: ``/^(`+)([^`].*)$/``
        
        Example: ```` ```p ````

     1. Backticks at the end of the _input character sequence_:
        ``/^(`+)$/``
        
        Example: ```` ``` ````

    In case of either pattern, the matching substring for the first
    parenthesized subexpression in the pattern is called the
    _opening-backticks-string_. The length of the
    _opening-backticks-string_ is called the _opening-backticks-count_.

    In case the match is with the first regular expression pattern, the
    matching substring for the second parenthesized subexpression in the
    pattern shall be called the _residual-code-span-sequence_. In case
    the match is with the second regular expression pattern, the
    _residual-code-span-sequence_ is said to be _null_.
    
 2. Set _code-content-length_ to 0

 3. If the _residual-code-span-sequence_ matches one of the following
    regular expression patterns:
    
     1. Non-backticks followed by backticks followed by a non-backtick:
        ``/^([^`]+)(`+)([^`].*)$/``

        Example: ````printf()```.````

     2. Non-backticks followed by backticks, at the end of the _input
        character sequence_: ``/^([^`]+)(`+)$/``

        Example: ````printf()``` ````

    then, the following is done:

    <!-- For some reason, Redcarpet requires a comment here to correctly
    display the following list -->

     1. The matching substring for the first parenthesized subexpression
        in the matching pattern is called the _code-fragment-string_.
        The matching substring for the second parenthesized
        subexpression in the matching pattern is called the
        _backticks-fragment-string_.

     2. The _code-content-length_ is incremented by the length of the
        _code-fragment-string_
        
     3. In case the match is with the first regular expression pattern,
        the _residual-code-span-sequence_ is set to the matching
        substring for the third parenthesized subexpression in the
        pattern. In case the match is with the second regular expression
        pattern, the _residual-code-span-sequence_ is set to _null_. 

     4. If the length of the _backticks-fragment-string_ is equal to the
        length of the _opening-backticks-count_, then the
        _backticks-fragment-string_ is identified as the
        _closing-code-string_

     5. If the length of the _backticks-fragment-string_ is not equal to
        the _opening-backticks-count_, then the _code-content-length_ is
        incremented by the length of the _backticks-fragment-string_

 4. If a _closing-code-string_ has not yet been identified, and if the
    _residual-code-span-sequence_ contains one or more `` ` ``
    characters, go to Step 3

 5. If a _closing-code-string_ has not yet been identified, the
    _opening-backticks-string_ is identified as a _text fragment_
 
 6. If a _closing-code-string_ has been identified, the following is
    done:

     1. Let _code-span-length_ be equal to 
        ( ( _opening-backticks-count_ * 2 ) + _code-content-length_ )
     2. The first _code-span-length_ characters of the
        [remaining-character-sequence] is identified as a _span tag
        candidate_, and interpreted as a **code span tag**
     3. Among the characters that form the _code span tag_, the first
        _opening-backticks-count_ characters and the last
        _opening-backticks-count_ characters are considered to be
        markup. The middle _code-content-length_ characters shall form
        the content of the code span. For HTML output, the content of
        the code-span should be _html-escaped_.

Thus, using this procedure, a sequence of one or more `` ` `` characters
at the [current-position] is identified to be the start of either a
_span tag candidate_ (of an _code-span tag_) or a _text fragment_.

<h4 id="handling-potential-image-tags">Handling potential image tags</h4>

[Handling potential image tags]: #handling-potential-image-tags

In this section, we discuss how to identify _span tags_ related to
images. This section assumes that the character at the
[current-position] is an unescaped `!` character, and that the immediate
next character is a `[` character.

<span id="image-tag-starter-pattern">The regular expression pattern
`/^!\[(([^\\\[\]]|\\.)*)(\].*)$/` is called the
**image-tag-starter-pattern**.</span>

Example: `![alt text` + _residual-image-sequence_

[image-tag-starter-pattern]: #image-tag-starter-pattern

If the [remaining-character-sequence] does not match the
[image-tag-starter-pattern], then the first 2 characters of the
[remaining-character-sequence] \(which should form the string `![`\) are
interpreted as a _text fragment_.

If the [remaining-character-sequence] matches the
[image-tag-starter-pattern], then:

 1. <span id="image-alt-text-string">The matching substring for the
    first parenthesized subexpression in the pattern is called the
    _image-alt-text-string_</span>
 2. <span id="residual-image-sequence">The matching substring for the
    last parenthesized subexpression in the pattern is called the
    _residual-image-sequence_</span>
 3. <span id="alt-text-pattern-match-length">The position at which the
    _residual-image-sequence_ starts within the
    [remaining-character-sequence] \(i.e. the number of characters
    present in the [remaining-character-sequence] before the start of
    the _residual-image-sequence_\) shall be called the
    _alt-text-pattern-match-length_</span>

[image-alt-text-string]: #image-alt-text-string
[residual-image-sequence]: #residual-image-sequence
[alt-text-pattern-match-length]: #alt-text-pattern-match-length

If the [remaining-character-sequence] matches the
[image-tag-starter-pattern], then the following is done:

 1. If the [residual-image-sequence] matches any of the following
    regular expression patterns:

     1. With trailing empty square brackets: `/^(\]\s*\[\s*\])/`

        Example: `][]`

     2. With no trailing opening bracket: `/^(\])\s*[^\[\(]/`

        Example: `] a`

     3. At the very end: `/^(\]\s*)$/`

        Example: `]`

    then the following is done:

    <!-- For some reason, Redcarpet requires a comment here to correctly
    display the following list -->

     1. The matching substring for the first and only
        parenthesized subexpression in the matching pattern shall be
        called the _image-ref-close-sequence_. The number of characters
        in the _image-ref-close-sequence_ is called the
        _image-ref-close-sequence-length_.

     2. Let _image-ref-tag-length_ be equal to the sum of the
        [alt-text-pattern-match-length] and the
        _image-ref-close-sequence-length_. The first
        _image-ref-tag-length_ characters of the
        [remaining-character-sequence] are collectively identified as a
        _span tag candidate_, and interpreted as an **image tag**.

     3. Let _reference id string_ be the string obtained on
        [simplifying] the [image-alt-text-string].

        The _reference id string_ shall be used to look up the actual
        image url and image title from the [link reference association
        map].

        If the [link reference association map] contains an entry for
        _reference id string_, then the output shall have the source of
        the image as the link url and the title of the image as the link
        title (if available) specified in the entry for the _reference
        id string_ in the [link reference association map]. The
        [image-alt-text-string] shall be used as the alternate text for
        the image. For HTML output, the title of the image and the
        alternate text for the image should be
        [attribute-value-escaped].
        
        If the [link reference association map] does not contain an
        entry for _reference id string_, then the output shall not
        include an image for this _image tag_. Instead, the first
        _image-ref-tag-length_ characters of the
        [remaining-character-sequence] are output as text. For HTML
        output, this text should be [html-escaped].

 2. If the [residual-image-sequence] matches the regular expression
    pattern `/^\]\s*\[(([^\\\[\]]|\\.)*)\]/` (Example: `] [ref id]`),
    then the following is done:

     1. The matching substring for the first parenthesized subexpression
        in the pattern is [simplified] to obtain the _reference id
        string_

     2. The length of the matching substring for the whole of the
        pattern is called the _image-ref-close-sequence-length_.

        Let _image-ref-tag-length_ be equal to the sum of the
        [alt-text-pattern-match-length] and the
        _image-ref-close-sequence-length_. The first
        _image-ref-tag-length_ characters of the
        [remaining-character-sequence] are collectively identified as a
        _span tag candidate_, and interpreted as an **image tag**.

     3. The _reference id string_ shall be used to look up the actual
        image url and image title from the [link reference association
        map].

        If the _link reference association map_ contains an entry for
        _reference id string_, then the output shall have the source of
        the image as the link url and the title of the image as the link
        title (if available) specified in the entry for the _reference
        id string_ in the [link reference association map]. The
        [image-alt-text-string] shall be used as the alternate text for
        the image. For HTML output, the title of the image and the
        alternate text for the image should be
        [attribute-value-escaped].
        
        If the [link reference association map] does not contain an
        entry for _reference id string_, then the output shall not
        include an image for this _image tag_. Instead, the first
        _image-ref-tag-length_ characters of the
        [remaining-character-sequence] are output as text. For HTML
        output, this text should be [html-escaped].

 3. If the [residual-image-sequence] matches the regular expression
    pattern `/^\]\s*\(/` and if both the following conditions are
    satisfied:

     1. The [residual-image-sequence] matches one of the following
        regular expression patterns:

         1. URL without angle brackets: `/^\]\s*\(\s*([^\(\)<>\s]+)([\)\s].*)$/`

            Example: `] (http://www.example.net/image.jpg` + _residual-image-attribute-sequence_

         2. URL within angle brackets: `/^\]\s*\(\s*<([^<>]*)>([\)].+)$/`
   
            Examples:  
            `](<http://example.net/image.jpg>` + _residual-image-attribute-sequence_  
            `] ( <http://example.net/image(1).jpg>` + _residual-image-attribute-sequence_

        In case of either pattern, the matching substring for the first
        parenthesised subexpression shall be called the _unprocessed
        image source string_, and the matching substring for the second
        parenthesized subexpression shall be called the
        _residual-image-attribute-sequence_. Any [whitespace]
        characters in the _unprocessed image source string_ are removed,
        and the resultant string is called the _image url string_.

        The position at which the _residual-image-attribute-sequence_
        starts within the _residual-image-sequence_ \(i.e. the
        number of characters present in the
        _residual-image-sequence_ before the start of the
        _residual-link-attribute-sequence_\) shall be called the
        _image-source-pattern-match-length_.

     2. The _residual-image-attribute-sequence_ matches one of the
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
            `"A (nice) \"title\" for the image")`  
            `'Title' followed by random ignored text)`  
            `"Title" followed by random "(ignored)" text)`  
            `just random ignored text)`  
            `just ignored text with \(escaped parentheses\))`

            If this is the matching pattern, the matching substring for
            the first parenthesized subexpression in the pattern is
            [trimmed] to give the _attributes-string_. If the
            _attributes-string_ begins with a [quoted string], then the
            [enclosed string] of the [quoted string] is called the
            _title string_. If the _attributes-string_ does not begin
            with a [quoted string], then the _title string_ is said to
            be _null_.

        The number of characters in the
        _residual-image-attribute-sequence_ that were consumed in
        matching the whole of the matching pattern is called the
        _image-attributes-pattern-match-length_.

    then, the following is done:

    <!-- For some reason, Redcarpet requires a comment here to correctly
    display the following list -->

     1. Let _image-src-tag-length_ be equal to the sum of
        [alt-text-pattern-match-length] and
        _image-source-pattern-match-length_ and
        _image-attributes-pattern-match-length_.  The first
        _image-src-tag-length_ characters of the
        [remaining-character-sequence] are collectively identified as a
        _span tag candidate_, and interpreted as an **image tag**.

     2. The _image url string_ shall be used as the source of the image.
        If _title string_ is not null, the _title string_ shall be used
        as the title of the image. The [image-alt-text-string] shall be
        used as the alternate text for the image. For HTML output, the
        title of the image and the alternate text for the image should
        be [attribute-value-escaped].

 4. If none of the above 3 conditions are satisfied, then the first 2
    characters of the [remaining-character-sequence] \(which should form
    the string `![`\) are interpreted as a _text fragment_.

Thus, using this procedure, the `![` sequence at the [current-position]
is identified to be the start of either a _span tag candidate_ (of an
_image tag_) or a _text fragment_.


<h4 id="handling-html-tags">Handling HTML tags</h4>

[Handling HTML tags]: #handling-html-tags

In this section, we discuss how to identify _span tags_ related to
inline HTML. This section assumes that the character at the
[current-position] is an unescaped `<` character.

Let _html-tag-detection-sequence_ be the [remaining-character-sequence].

To identify _span tags_ related to inline HTML we need to employ the use
of a HTML parser. We supply characters in the
_html-tag-detection-sequence_ to the HTML parser, one character at a
time, till one of the following happens:

 1. The HTML parser detects a complete self-closing HTML tag (this also
    includes tags empty by definition in HTML4, like `<br>` and `<img
    src="">`)

    If this happens first, the following is done:
    
     1. The text that represents the opening HTML tag is identified as a
        _span tag candidate_, and identified as a **self-closing HTML
        tag**.

     2. For HTML output, the text that represents the self-closing HTML
        tag shall be included in the output verbatim.

 2. The HTML parser detects a complete opening HTML tag.

    If this happens first, the following is done:

     1. The text that represents the opening HTML tag is identified as a
        _span tag candidate_, and interpreted as an **opening HTML tag**
    
     2. A new node is pushed onto the [stack of potential opening span
        tags] with the following [properties][stack-node-properties]:

         1. The _tag string_ of the node is set to the text that
            represents the opening HTML tag
         2. The _node type_ of the node is set as _raw html node_
         3. The _html tag name_ of the node is set to the HTML tag name
            of the opening HTML tag that was just identified

     3. For HTML output, the text that represents the opening HTML tag
        shall be included in the output verbatim.

 3. The HTML parser detects a complete closing HTML tag.

    If this happens first, the following is done:

     1. The text that represents the closing HTML tag is identified as a
        _span tag candidate_, and interpreted as a **closing HTML tag**
    
     2. Let _currently open html node_ be the [topmost node of type]
        _raw html node_

     3. If the _currently open html node_ is not _null_, and the _html
        tag name_ of the _currently open html node_ is the same as the
        HTML tag name of the closing HTML tag that was just identified,
        the [top node] shall be popped off

     4. If the _currently open html node_ is null, or if  _html tag
        name_ of the _currently open html node_ is not the same as the
        HTML tag name of the closing HTML tag that was just identified,
        then all nodes in the [stack of potential opening span tags]
        whose _node type_ is not equal to _raw html node_ shall be
        removed from the stack and interpreted as _text fragments_

     5. For HTML output, the text that represents the closing HTML tag
        shall be included in the output verbatim.

 4. The HTML parser detects a complete HTML comment.

    If this happens first, the text that represents the HTML comment is
    identified as a _span tag candidate_, and interpreted as a **comment
    HTML tag**.

    For HTML output, the text that represents the comment HTML tag shall
    be included in the output verbatim.

 5. The HTML parser detects HTML text, or the HTML parser detects an
    error, or the _html-tag-detection-sequence_ has no more characters
    to supply to the HTML parser.

    If this happens first, the `<` at the [current-position] is
    identified as a _text fragment_.

Thus, using this procedure, the `<` at the [current-position] is
identified to be the start of either a _span tag candidate_ or a _text
fragment_. The _span tag candidate_ is also interpreted as one of these
_span tags_: _opening HTML tag_ or _closing HTML tag_ or _self-closing
HTML tag_ or _comment HTML tag_.

