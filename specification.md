---

layout: doc  
title: "The vfmd specification"  
permalink: specification/  
license: http://vfmd.github.io/vfmd-spec/LICENSE.txt  
license-link-text: "an MIT-style license"  

---

# The vfmd specification

**vfmd** is a variant of [Markdown] with an unambiguous specification of
its syntax.

This document describes the specification for the vfmd syntax. It is
intended to be read by someone implementing this specification to parse
or otherwise programmatically interpret vfmd input. If you only intend
to write a document using vfmd, please read the [userguide] instead.

[Markdown]: http://daringfireball.net/projects/markdown/
[vfmd]: introduction.md
[userguide]: http://vfmd.github.io/vfmd-spec/userguide/

<h2>Table of contents</h2>

  * [About this specification]
    * [Scope of this specification]
    * [Structure of this specification]
    * [Conventions]
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
    * [Procedure for identifying span tags]
    * [Procedure for identifying link tags]
    * [Procedure for identifying emphasis tags]
    * [Procedure for identifying code-span tags]
    * [Procedure for identifying image tags]
    * [Procedure for detecting automatic links]
    * [Procedure for identifying HTML tags]
  * [Additional processing]
    * [De-escaping]
    * [Processing text fragments]
    * [Processing for HTML output]
  * [Extending the syntax]

<h2 id="about-this-spec">About this specification</h2>

[About this specification]: #about-this-spec

<h3 id="scope-of-this-specification">Scope of this specification</h3>

[Scope of this specification]: #scope-of-this-specification

This specification defines how a vfmd source document should be
interpreted. Wherever special consideration is required in creating a
HTML representation of the document, this specification also specifies
the required HTML output. Special considerations that may be required
for other output formats is not explicitly defined in this
specification.

<h3 id="structure-of-this-specification">Structure of this specification</h3>

[Structure of this specification]: #structure-of-this-specification

The specification is divided into the following major sections:

  * [About this specification]: Defines terminologies and conventions
    used in this specification.

  * [Identifying block-elements]: Defines how block-level syntax
    elements should be recognized.

  * [Interpreting block-elements]: Defines how the block-level
    constructs, after being recognized, should be handled.

  * [Identifying span-elements]: Defines how span-level syntax elements
    should be recognized and handled.

  * [Additional processing]: Describes the processing that needs to be
    done on the non-markup parts of the document.

  * [Extending the syntax]: Shows how this specification can be extended
    to support extensions to the core vfmd syntax.

<h3 id="conventions">Conventions</h3>

[Conventions]: #conventions

<h4 id="regular-expression-conventions">Regular expression conventions</h4>

[Regular expression conventions]: #regular-expression-cpnventions

This specification makes use of regular expressions to define the
syntax. A regular expression appears as `/regularexpression/` in this
specification - i.e. it appears in a code-span, enclosed between two
forward slashes (like in Perl code).

The regular expressions follow the [PCRE syntax] in UTF mode.
Specifically, the following should be considered when reading the
regular expressions in this document:

 1. A `\` character is used to escape any special character within a
    regular expression, irrespective of whether it has a special meaning
    at that position or not
 2. A <code style="white-space: pre;"> </code> within a regular
    expression indicates a [space] character
 3. A `\s` within a regular expression indicates a [whitespace]
    character

The regular expressions used in this specification do not use any
extended regular expression syntax (e.g. min/max quantifiers,
backreferences, etc.), and confirm to a regular grammar. This means that
these regular expressions can be adapted to any other regular expression
syntax as may be required for an implementation of this specification.

In this specification, when it is said that a string matches a regular
expression, it is used in the meaning that a part or whole of the string
matches the whole of the regular expression. Whenever the whole of the
string needs to match the whole of the regular expression, that
requirement is made explicit in the regular expression by starting it in
`^` and ending it in `$`.

[PCRE syntax]: http://man.he.net/man3/pcrepattern

<h3 id="definitions">Definitions</h3>

[Definitions]: #definitions

<h4 id="document">Document</h4>

[document]: #document

The input Markdown text is called the **document**.

The [document] contains Unicode text in UTF-8 encoding without any
leading Byte-Order-Mark. Any byte sequences in the input that are
invalid in UTF-8 encoding are filtered off and are ignored. Therefore,
for the following discussion, the [document] is considered to not have
any invalid byte sequences.

<h4 id="characters">Characters</h4>

[character]: #characters
[characters]: #characters

A **character** is an atomic unit of text specified as a Unicode code
point and encoded in UTF-8 encoding.

The [document] consists of a sequence of [characters], where the
[characters] may represent either markup or character data.

A U+0009 (TAB) character in the input shall be treated as four
consecutive U+0020 (SPACE) characters.

<span id="space">
A U+0020 (SPACE) character is henceforth called a **space** character.
</span>
<span id="non-space">
Any character that is not a U+0020 (SPACE) character is henceforth
called a **non-space** character.
</span>

The character sequence of a U+000D (CR) character followed by a U+000A
(LF) character in the input shall be treated as a single U+000A (LF)
character.

<span id="line-break">
A U+000A (LF) character is called a **line break** character.
</span>
<span id="non-line-break">
Any [character] that is not a _line break_ character is called a
**non-line-break** character.
</span>

<span id="whitespace">
A **whitespace** character is one of the following characters:
U+0009 (TAB), U+000A (LF), U+000C (FF), U+000D (CR) or U+0020 (SPACE)
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

<h4 id="lines">Lines</h4>

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

To [identify the block-elements] in the [document], the [document] is seen
as a sequence of [lines].

[blank line]: #blank-line
[blank lines]: #blank-line

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

<span id="parent-line-sequence">When a _block-element line sequence_ is
obtained by breaking up an [input line sequence], the [input line
sequence] is said to be the _parent line sequence_ of the _block-element
line sequence_.</span>

[block-element start line]: #block-element-line-sequence
[block-element end line]: #block-element-line-sequence
[parent line sequence]: #parent-line-sequence

<h3 id="type-and-extent-block-element">Type and extent of a block-element</h3>

[Type and extent of a block-element]: #type-and-extent-block-element

The type of the block-element is determined based on the [block-element
start line] and, in some cases, the line following the [block-element
start line].  The line at which the block should end (i.e.  the
corresponding [block-element end line]) is determined based on the
[block-element start line] and subsequent lines.

We define the following regular expression patterns:

 * <span id="unordered-list-starter-pattern">
   **unordered list starter pattern**</span>: `/^( *[\*\-\+] +)[^ ]/`

   Example: <code style="white-space: pre;">  * </code>

 * <span id="ordered-list-starter-pattern">
   **ordered list starter pattern**</span>: `/^( *([0-9]+)\. +)[^ ]/`

   Example: <code style="white-space: pre;"> 1. </code>

 * <span id="horizontal-rule-pattern">
   **horizontal rule pattern**</span>: `/^ *((\* *\* *\* *[\* ]*)|(\- *\- *\- *[\- ]*)|(_ *_ *_ *[_ ]*))$/`

   Any line that matches the _horizontal rule pattern_ would be
   composed of just asterisks (minimum three) and [space] characters, or
   just underscores (minimum three) and [space] characters, or just
   dashes (minimum three) and [space] characters.

   Examples:  
   `*****`  
   `   -- -- --   `

[unordered list starter pattern]: #unordered-list-starter-pattern
[ordered list starter pattern]: #ordered-list-starter-pattern
[horizontal rule pattern]: #horizontal-rule-pattern

The following rules are to be followed in determining the type of the
block-element and the [block-element end line]:

 1. If the [block-element start line] is a [blank line], then the body
    element is of type [**null block**]. The same line is the
    [block-element end line].

 2. If the [block-element start line] does not begin with four or more
    consecutive [space] characters, and if the [block-element start
    line] matches one of the following regular expression patterns:

     1. Label and URL without angle brackets:  
        `/^ *\[([^\\\[\]]|\\.)*\] *: *[^ <>]+( .*)?$/`

        Example: `[ref id]: url` + _ref-definition-trailing-sequence_

     2. Label and URL within angle brackets:  
        `/^ *\[([^\\\[\]]|\\.)*\] *: *<[^<>]*>(.*)$/`

        Example: `[ref id]: <url>` + _ref-definition-trailing-sequence_

    then the block-element is of type [**reference-resolution block**].
    The matching substring for the last parenthesized subexpression in
    the matching pattern shall be called the
    _ref-definition-trailing-sequence_.

    If all the following conditions are satisfied:

    <!-- For some reason, Redcarpet requires a comment here to correctly
    display the following list -->

     1. The _ref-definition-trailing-sequence_ does not contain any
        [non-space] characters
     2. The [block-element start line] is not the last line in the
        [input line sequence]
     3. The [block-element start line] is immediately followed by a
        succeeding line that matches the regular expression pattern
        `/^ +("(([^"\\]|\\.)*)"|'(([^'\\]|\\.)*)'|\(([^\\\(\)]|\\.)*\)) *$/`

        Examples:  
        <code style="white-space: pre;"> "Title"</code>   
        <code style="white-space: pre;"> (Title)</code>  
        <code style="white-space: pre;">   'A "title" in single quotes'</code>  
        <code style="white-space: pre;"> "Title with \"quotes\""</code>

    then the [block-element end line] is the line that immediately
    follows the [block-element start line]; else, the [block-element end
    line] is the same line as the [block-element start line].

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
    character, then the block-element is of type [**blockquote**].
    The [block-element end line] is the next subsequent line in the
    [input line sequence], starting from and inclusive of the
    [block-element start line], that satisfies one of the following
    conditions:

     1. The line is a [blank line] and is immediately succeeded by
        another [blank line]

        (or)

     2. The line is a [blank line] and is immediately succeeded by a
        succeeding line that begins with four or more consecutive [space]
        characters

        (or)

     3. The line is a [blank line] that is immediately succeeded by a
        succeeding line, and the first [non-space] character in the
        succeeding line is not a `>` character

        (or)

     4. The line is not a [blank line] and is immediately succeeded by a
        succeeding line that statisfies all of the following conditions:
         1. The succeeding line does not begin with four or more
            consecutive [space] characters
         2. The succeeding line matches the [horizontal rule pattern]

    If no such [block-element end line] is found, the last line in the
    [input line sequence] is the [block-element end line].

 7. If none of the above conditions apply, and if the [block-element
    start line] matches the [horizontal rule pattern], then the line
    forms a block-element of type [**horizontal rule**].

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
         3. The succeeding line does not begin with four or more
            consecutive [space] characters
         4. The succeeding line matches the [unordered list
            starter pattern], or matches the [ordered list starter
            pattern], or matches the [horizontal rule pattern]

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
         3. The succeeding line does not begin with four or more
            consecutive [space] characters
         4. The succeeding line either matches the [unordered list
            starter pattern], or matches the [horizontal rule pattern]

    If no such [block-element end line] is found, the last line in the
    [input line sequence] is the [block-element end line].

 <!-- Redcarpet doesn't like double-digit line numbers at this point,
      so using 0 instead of 10 -->

 0. If none of the above conditions apply, then the [block-element start
    line] marks the start of a block-element of type [**paragraph**].

    In order to find the [block-element end line], we need to make use
    of a HTML parser. To the HTML parser, we feed the characters of each
    line, starting from the [block-element start line]. After feeding
    all characters of every line, we feed a [line break] character to
    the HTML parser, and observe the state of the HTML parser. Of the
    the many possible states of a HTML parser, we are only interested in
    the [HTML parser states relevant to finding the end of a paragraph].

    The [block-element end line] is the next subsequent [line] in the
    [input line sequence], starting from and inclusive of the
    [block-element start line], that satisfies all the following
    conditions:

     1. At the end of feeding the line and a [line break] to the HTML
        parser, all the following conditions are satisfied:

         1. The HTML parser state is not "within a HTML tag"

         2. The HTML parser state is not "within a HTML comment"

         3. The HTML parser state is not "within the contents of a
            well-formed [verbatim HTML element]"

     2. The line is a [blank line], or is immediately succeeded by a
        succeeding line that does not begin with four or more
        consecutive [space] characters, and satisfies at least one of
        the following conditions:

         1. The succeeding line matches the [horizontal rule pattern]

            (or)

         2. The leftmost [non-space] character in the succeeding line is
            a `>` character, and the [input line sequence] is a
            [blockquote-processed line sequence]

            (or)

         3. The succeeding line matches the [ordered list starter
            pattern], and the [input line sequence] is a
            [list-item-processed line sequence]

            (or)

         4. The succeeding line matches the [unordered list starter
            pattern], and the [input line sequence] is a
            [list-item-processed line sequence]


    If no such [block-element end line] is found, the last line in the
    [input line sequence] is the [block-element end line].

Using the above rules, the [input line sequence] is broken down into a
series of [block-element line sequences], and the block-element type of
each [block-element line sequence] is identified.

<span id="block-level-extensions">An implementation can extend the core
vfmd syntax to support additional syntax elements. For every additional
block-level syntax element, the implementation shall insert a rule in
the above rule list, at a position at which it makes sense to recognize
the particular construct. Given a [block-element start line], the rule
shall identify whether the line is the beginning of the said block-level
construct, and shall determine till what line in the [input line
sequence] the block-level construct extends. Doing so will identify the
[block-element line sequence] for the block-level construct. The
implementation can decide how the [block-element line sequence] should
be interpreted and output.</span>

[verbatim HTML element]: #verbatim-html-element
[block-level extensions]: #block-level-extensions
[block-level extension]: #block-level-extension

<h4 id="html-parser-states-relevant-to-end-of-paragraph">
HTML parser states relevant to finding the end of a paragraph</h4>

[HTML parser states relevant to finding the end of a paragraph]: #html-parser-states-relevant-to-end-of-paragraph

When looking for the [block-element end line] for a block-element of
type [paragraph], we make use of a HTML parser. Of the the many possible
states of a HTML parser, we are interested in only some of the states.
This section enumerates and describes the parser states that are of
interest in this context.

<span id="verbatim-html-element">We define a **verbatim HTML element**
to be one of these HTML elements: `pre`, `script`, `tag`. We define a
**non-verbatim HTML element** to be any HTML element other than a
_verbatim HTML element_.</span>

[verbatim HTML element]: #verbatim-html-element
[non-verbatim HTML element]: #verbatim-html-element

The relevant states of the HTML parser when looking for the
[block-element end line] for a block-element of type [paragraph] are:

 1. Within a HTML tag (open / close / self-closing tag)

    For example, this is the state at the end of the first line below:

        <div id="div1"
        >

 2. Within a HTML comment

    For example, this is the state at the end of the first line below:

        <!-- Insert illuminating comment here
        -->

 3. Within the contents of a well-formed [verbatim HTML element]

    For example, this is the state at the end of the first line below:

        This open <pre> tag has a
        corresponding close </pre> tag

 4. Within the contents of a not-well-formed [verbatim HTML element]
    \(i.e. after the open tag of an unclosed or not-properly-closed
    [verbatim HTML element]\)

    For example, this is the state at the end of the first line below:

        This open <pre> tag does not have a
        corresponding close tag in the document


 5. Within the contents of a [non-verbatim HTML element] \(well-formed
    or not\)

    For example, this is the state at the end of the first line below:

        Outside <div> Inside
        Inside </div> Outside


 6. Outside of any HTML element or comment

    For example, this is the state at the end of the first line below:

        Outside <div> Inside </div> Outside
        Outside

Note that a HTML parser can know whether a `<` marks the start of a HTML
construct or not only after it has seen the rest of the text. For
example, if a matching `>` is not found, the first `<` does not indicate
the start of a HTML construct at all. As another example, if the `<` is
immediately followed by a `!--` and later followed by a `-->`, the `<`
marks the start of a HTML comment. Consequently, the HTML parser should
lookahead as many character as may be necessary and return the
appropriate state considering all these possibilities.

Also note that a HTML parser can know whether a HTML element is
well-formed or not, only after encountering an end tag. So, after
consuming only part of the input, the HTML parser might not know whether
it's in the "within the contents of a well-formed [verbatim HTML
element]" state, or it's in the "within the contents of a
not-well-formed [verbatim HTML element]" state. So, in order to find the
end of the paragraph without backtracking, it is suggested that an
implementation employ the following design:

  * When an opening tag of a [verbatim HTML element] is encountered,
    keep in mind that it could turn out to be either a well-formed HTML
    element, or a not-well-formed HTML element
  * Keep track of what the [block-element end line] for the paragraph
    would have been in either case
  * When it becomes clear whether the HTML element is well-formed or
    not, pick the appropriate choice for the [block-element end line]

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
    interpreting the _header text run_ as a [text span sequence] shall
    form the content of the header element.

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
result of interpreting the _header text run_ as a [text span sequence]
shall form the content of the header element.

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

<span id="blockquote-processed-line-sequence">
Each [line] in the [block-element line sequence] is processed to produce
a modified sequence of [lines], called the _blockquote-processed line
sequence_. The following processing is to be done for each [line]:
</span>

 1. If the [line] matches the regular expression `/^ *> /`, then the
part of the [line] that matches the said regular expression shall be removed
from the line
 2. If the pattern in (1) above is not satisfied, and if the [line] matches
the regular expression `/^ *>/`, then the part of the [line] that
matches the said regular expression shall be removed from the line

The [blockquote-processed line sequence] obtained this way can be
considered as the [input line sequence] for a sequence of block-elements
nested within the blockquote. The result of interpreting that [input
line sequence] further into block-elements shall form the content of the
blockquote element.

[blockquote-processed line sequence]: #blockquote-processed-line-sequence

For example, consider the following [block-element line sequence]:

      > In Perl, a Hello World is
      > written as follows:
      >
      >     print "Hello World!\n";

After processing each line in the above [block-element line sequence],
the [blockquote-processed line sequence] obtained is as follows:

    In Perl, a Hello World is
    written as follows:

        print "Hello World!\n";

When we treat the [blockquote-processed line sequence] as an [input line
sequence], we can recognize nested block elements in it of type
paragraph and code block. The HTML equivalent for the [lines] in the
[blockquote-processed line sequence] is as follows:

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

<span id="unordered-list-item-line-sequence">
We first divide the [block-element line sequence] into a series of
_unordered list item line sequences_. The lines in a particular
_unordered list item line sequence_ correspond to one list item in the
list.</span>

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

<span id="unordered-list-item-processed-line-sequence">
Each [line] in the [unordered list item line sequence] is processed to
produce a modified sequence of [lines], called the
_unordered-list-item-processed line sequence_. The following processing
is to be done for each [line]:</span>

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

Each [unordered-list-item-processed line sequence] obtained this way can
be considered as the [input line sequence] for a sequence of
block-elements nested within the list item. The result of interpreting
that [input line sequence] further into block-elements shall form the
content of the list element.

An [unordered-list-item-processed line sequence] has certain
[properties](#properties-of-list-item-line-sequences) that are useful in
determining how the [paragraph] block-elements (if any) contained within
the [unordered-list-item-processed line sequence] should be handled.

The list elements so obtained are combined into a sequence to form the
complete unordered list in the output.

[unordered list item line sequence]: #unordered-list-item-line-sequence
[unordered list item line sequences]: #unordered-list-item-line-sequence
[unordered-list-item-processed line sequence]: #unordered-list-item-processed-line-sequence
[unordered-list-item-processed line sequences]: #unordered-list-item-processed-line-sequence

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

<span id="ordered-list-item-line-sequence">
We first divide the [block-element line sequence] into a series of
_ordered list item line sequences_. The lines in a particular
_ordered list item line sequence_ correspond to one list item in the
list.</span>

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

<span id="ordered-list-item-processed-line-sequence">
Each [line] in the [ordered list item line sequence] is processed to
produce a modified sequence of [lines], called the
_ordered-list-item-processed line sequence_. The following processing is
to be done for each [line]:</span>

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

Each [ordered-list-item-processed line sequence] obtained this way can be
considered as the [input line sequence] for a sequence of block-elements
nested within the list item. The result of interpreting that [input line
sequence] further into block-elements shall form the content of the list
element.

An [ordered-list-item-processed line sequence] has certain
[properties](#properties-of-list-item-line-sequences) that are useful in
determining how the [paragraph] block-elements (if any) contained within
the [ordered-list-item-processed line sequence] should be handled.

The list elements so obtained are combined into a sequence to form the
complete ordered list in the output.

The numbering for the ordered list should start from the _ordered list
starting number_. For HTML output, if the _ordered list starting number_
is the number '1', the corresponding `ol` start tag in the output shall
not have the `start` attribute; if the _ordered list starting number_ is
not the number '1', the corresponding `ol` start tag in the output shall
include the `start` attribute with the the _ordered list starting
number_ as the attribute value.

[ordered list item line sequence]: #ordered-list-item-line-sequence
[ordered list item line sequences]: #ordered-list-item-line-sequence
[ordered-list-item-processed line sequence]: #ordered-list-item-processed-line-sequence
[ordered-list-item-processed line sequences]: #ordered-list-item-processed-line-sequence

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
The resulting sequence of [characters] is trimmed to give the _paragraph text_.
The result of interpreting the _paragraph text_ as a [text span
sequence] shall form the content of the paragraph element.

<span id="phrasing-html-element"> We define a **phrasing-html-element**
to be one of the following HTML elements:</span> `a`, `abbr`, `area`,
`audio`, `b`, `bdi`, `bdo`, `br`, `button`, `canvas`, `cite`, `code`,
`data`, `datalist`, `del`, `dfn`, `em`, `embed`, `i`, `iframe`, `img`,
`input`, `ins`, `kbd`, `keygen`, `label`, `map`, `mark`, `meter`,
`noscript`, `object`, `output`, `progress`, `q`, `ruby`, `s`, `samp`,
`select`, `small`, `span`, `strong`, `sub`, `sup`, `textarea`, `time`,
`u`, `var`, `video` or `wbr`. These are elements in the HTML namespace
that belong to the [phrasing content] category in [HTML5].

[HTML5]: http://www.w3.org/TR/html5/ "HTML5 Specification"
[phrasing content]: http://www.w3.org/TR/html5/dom.html#phrasing-content-1
[phrasing-html-element]: #phrasing-html-element

For HTML output, the _paragraph text_ needs to be run through a HTML
parser to determine how the content of the paragraph element should be
presented.

For HTML output, the content of the paragraph element shall be enclosed
in `p` tags, unless any of the following conditions is satisfied:

 1. The _paragraph text_ contains an unmatched HTML tag (i.e. open tag
    without a close tag, or a close tag without an open tag)

 2. The _paragraph text_ contains a misnested HTML tag (i.e. close tag
    at the wrong position)

 3. The _paragraph text_ contains a HTML element that is not a
    [phrasing-html-element]

 4. The _paragraph text_ contains a HTML comment

 5. The [block-element line sequence] for the paragraph block is the
    first [block-element line sequence] of its [parent line sequence],
    and the [parent line sequence] is a
    [top-packed list-item-processed line sequence]

 6. The [block-element line sequence] for the paragraph block is the
    last [block-element line sequence] of its [parent line sequence],
    but not the second [block-element line sequence] of its
    [parent line sequence], and the [parent line sequence] is a
    [bottom-packed list-item-processed line sequence]

If one or more of the above 6 conditions is satisfied, the HTML output
of the paragraph shall be the same as the content of the paragraph,
without wrapping it in `p` tags.

<h3 id="reference-resolution-block">reference-resolution block</h3>

[reference-resolution block]: #reference-resolution-block
[**reference-resolution block**]: #reference-resolution-block

The [block-element line sequence] for a reference-resolution block shall
have either a single [line] or two [lines].

The reference-resolution block does not result in any output by itself.
It is used to resolve the URLs of reference-style links and images in
the rest of the document.

In case the [block-element line sequence] contains a single [line], that
[line] is called the _link-definition-line_.

In case the [block-element line sequence] contains two [lines], the
second [line] is appended to the first [line] to produce the
_link-definition-line_.

The _link-definition-line_ shall not contain any [line break]
characters.

The _link-definition-line_ shall match one of the following regular
expression patterns:

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
    `[ref    id]: http://example.net/ (Title)`  
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
pattern is called the _title container string_.

If the _title-container-string_ matches the regular expression pattern
`/^\((([^\\\(\)]|\\.)*)\)/`, then the matching substring for the first
(i.e. outer) parenthesized subexpression in the pattern is called the
_link title string_.

If the _title-container-string_ begins with a [quoted string], the
[enclosed string] of the [quoted string] is called the _link title
string_ and the rest of the _title-container-string_ is ignored.

The _reference id string_ is said to be associated with the _link url
string_ and the _link title string_ (if a _link title string_ was
found). A new entry is added to the [link reference association map]
with the _reference id string_ as the key, and the _link url string_ and
the _link title string_ as values, unless the [link reference
association map] already has an entry with the _reference id string_ as
the key.

The <span id="link-reference-association-map">**link reference
association map**</span> is an associative array that contains data from
all the reference-resolution blocks in the document, including those
that occur within blockquotes and lists. It helps in
mapping a _reference id_ to the _link url_ and _link title_ that the
_reference id_ represents. It is used in the [procedure for identifying
link tags] and in the [procedure for identifying image tags] to resolve
a reference id to a link url and, if available, a link title. All
lookups in the _link reference association map_ are made
case-insensitively.

[link reference association map]: #link-reference-association-map

<h3 id="null-block">null block</h3>

[null block]: #null-block
[**null block**]: #null-block

The  [block-element line sequence] for a null block element shall have a
single [blank line].

A null block does not result in any output.

<h3 id="properties-of-list-item-line-sequences">Properties of list item line sequences</h3>

[Properties of list item line sequences]: #properties-of-list-item-line-sequences

<span id="list-item-line-sequence">
A _list item line sequence_ denotes either an [unordered list item line
sequence] or an [ordered list item line sequence].</span>

<span id="list-item-processed-line-sequence"> A _list-item-processed
line sequence_ denotes either an [unordered-list-item-processed line
sequence] or an [ordered-list-item-processed line sequence].</span>

[list item line sequence]: #list-item-line-sequence
[list-item-processed line sequence]: #list-item-processed-line-sequence

<h4 id="top-packed-list-item-line-sequence">
Top-packed list item line sequences</h4>

[Top-packed list item line sequence]: #top-packed-list-item-line-sequence
[top-packed list item line sequence]: #top-packed-list-item-line-sequence
[top-packed list-item-processed line sequence]: #top-packed-list-item-line-sequence

A [list item line sequence], _S_, is said to be _top-packed_ if, and
only if, _S_ satisfies any of the following conditions:

 1. _S_ is the only [list item line sequence] in the
    [block-element line sequence]

    (or)

 2. _S_ is the first [list item line sequence] in the
    [block-element line sequence], and the last line of _S_ is
    not a [blank line]

    (or)

 3. _S_ is not the first [list item line sequence] in the [block-element
    line sequence], and the line immediately before the first line of
    _S_ in the [block-element line sequence] is not a [blank line]

If a [list item line sequence] is _top-packed_, the
[list-item-processed line sequence] obtained from it is also said to be
_top-packed_. Otherwise, the [list-item-processed line sequence] is not
said to be _top-packed_.

<h4 id="bottom-packed-list-item-line-sequence">
Bottom-packed list item line sequences</h4>

[Bottom-packed list item line sequence]: #bottom-packed-list-item-line-sequence
[bottom-packed list item line sequence]: #bottom-packed-list-item-line-sequence
[bottom-packed list-item-processed line sequence]: #bottom-packed-list-item-line-sequence

A [list item line sequence], _S_, is said to be _bottom-packed_ if, and
only if, _S_ satisfies any of the following conditions:

 1. _S_ is the only [list item line sequence] in the [block-element line
    sequence]

    (or)

 2. The last line of _S_ is not a [blank line]

    (or)

 3. _S_ is the last [list item line sequence] in the [block-element line
    sequence], and the line immediately before the first line of _S_ in
    the [block-element line sequence] is not a [blank line]

If a [list item line sequence] is _bottom-packed_, the
[list-item-processed line sequence] obtained from it is also said to be
_bottom-packed_. Otherwise, the [list-item-processed line sequence] is
not said to be _bottom-packed_.

<h2 id="identifying-span-elements">Identifying span-elements</h2>

[Identifying span-elements]: #identifying-span-elements
[text span sequence]: #identifying-span-elements
[input character sequence]: #identifying-span-elements

A **text span sequence** is a sequence of span-level vfmd constructs in
a paragraph or header block.

To interpret a non-empty sequence of characters, called the **input
character sequence**, as a _text span sequence_, we need to identify the
type and extent of the span-elements in the input.

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

<span id="topmost-node-of-type-is-null">The _topmost node_ of type _t_
is said to be _null_ if, and only if, any of the following conditions is
true:</span>

 1. The [stack of potential opening span tags] does not contain any node
    whose _node type_ is equal to _t_

    (or)

 2. All nodes whose _node type_ is equal to _t_ in the [stack of
    potential opening span tags] have a _html node_ above them (where
    _html node_ means a node whose _node type_ is _raw html node_)

To identify and interpret the _span tags_ in the [input character
sequence], the [procedure for identifying span tags] shall be used.

[stack of potential opening span tags]: #stack-of-potential-opening-span-tags
[stack-node-properties]: #stack-node-properties
[top node]: #top-node
[topmost node of type]: #topmost-node-of-type

<h3 id="procedure-for-identifying-span-tags">
Procedure for identifying span tags</h3>

[Procedure for identifying span tags]: #procedure-for-identifying-span-tags
[procedure for identifying span tags]: #procedure-for-identifying-span-tags

In this section, we discuss the procedure to identify and interpret the
_span tags_ in the [input character sequence].

The procedure involves iterating over the characters in the [input
character sequence]. <span id="current-position">The current character
position in the [input character sequence] is called the
**current-position**.</span> When _current-position_ is 0, the first
character in the [input character sequence] is said to be the character
at the _current-position_; when _current-position_ is 1, the second
character in the [input character sequence] is said to be the character
at the _current-position_, and so on. <span
id="remaining-character-sequence">The substring of the [input character
sequence] starting from and including the character at the
_current-position_ and ending at the end of the [input character
sequence] is called the **remaining-character-sequence**.</span> When
_current-position_ is 0, the _remaining-character-sequence_ is equal to
the [input character sequence].

The root procedure in turn invokes other procedures. In the method
described here, global variables are used to communicate information
from invoked procedures back to the root procedure, but better means can
be adopted by an implementation. The global variables used are:

 1. <span id="consumed-character-count">_consumed-character-count_: The
    number of characters that were consumed to form a _span tag_ or a
    _text fragment_</span>
 2. <span id="is-verbatim-html-mode">_is-verbatim-html-mode_: A boolean
    that decides whether vfmd syntax should be recognized at this
    position in the document, or not</span>

[current-position]: #current-position
[remaining-character-sequence]: #remaining-character-sequence
[consumed-character-count]: #consumed-character-count
[is-verbatim-html-mode]: #is-verbatim-html-mode

The procedure to identify and interpret the _span tags_ is as follows:

 1. Set [current-position] as 0
 2. Set _is-verbatim-html-mode_ as _false_
 3. <span id="span-proc-step-3">Set _consumed-character-count_ as 0
    </span>
 4. If _is-verbatim-html-mode_ is equal to _false_, do the following:
     1. If the character at the [current-position] is either an
        unescaped `[` (open square bracket) character, or an unescaped
        `]` (close square bracket) character, invoke the [procedure for
        identifying link tags]
     2. If the character at the [current-position] is either an
        unescaped `*` (asterisk) character, or an unescaped `_`
        (underscore or low line) character, then invoke the [procedure
        for identifying emphasis tags]
     3. If the character at the [current-position] is an unescaped `` `
        `` (backtick) character, then invoke the [procedure for
        identifying code-span tags]
     4. If the character at the [current-position] is an unescaped `!`
        (exclamation mark) character, and if the
        [remaining-character-sequence] matches the regular expression
        pattern `/!\[/`, then invoke the [procedure for identifying
        image tags]
     5. If _consumed-character-count_ is equal to 0, then invoke the
        [procedure for detecting automatic links]
     6. If _consumed-character-count_ is equal to 0, and if the
        character at the [current-position] is an unescaped `<` (left
        angle bracket) character, then invoke the [procedure for
        identifying HTML tags]
     7. If _consumed-character-count_ is equal to 0, interpret the
        character at the [current-position] to be part of a _text
        fragment_, and set _consumed-character-count_ as 1
 5. If _is-verbatim-html-mode_ is equal to _true_, interpret the
    [remaining-character-sequence] as _verbatim-html_, and set
    _consumed-character-count_ to the length of the
    [remaining-character-sequence]
 6. Increment [current-position] by _consumed-character-count_
 7. If [current-position] is less than the length of the
    [input character sequence], go to [Step 3](#span-proc-step-3)

The _text fragments_ identified in the above procedure should be
handled as specified in [processing text fragments].

Any _verbatim-html_ identified in the above procedure should be output
verbatim, without subjecting it to the
[processing for text fragments][processing text fragments].

<span id="span-level-extensions">An implementation can extend the core
vfmd syntax to support additional syntax elements. For every additional
span-level syntax element, the implementation shall insert a rule in
the above rule list, at a position at which it makes sense to recognize
the particular construct. The rule shall determine whether a span-level
construct begins at a particular position in the [input character
sequence] or not, and if it does, how it should be interpreted and
output.</span>

[span-level extensions]: #span-level-extensions
[span-level extension]: #span-level-extensions

<h3 id="procedure-for-identifying-link-tags">Procedure for identifying link tags</h3>

[Procedure for identifying link tags]: #procedure-for-identifying-link-tags
[procedure for identifying link tags]: #procedure-for-identifying-link-tags

This procedure assumes that the character at the [current-position]
is either an unescaped `[` character or an unescaped `]` character.

If the character at the [current-position] is a `[` character, it
implies that the `[` can potentially get interpreted as an _opening link
tag_ in the future. In this case, the following is done:

 1. A new node is pushed onto the [stack of potential opening span
    tags] with the following [properties][stack-node-properties]:

     1. The _tag string_ of the node is set to the `[` character at
        the current position
     2. The _node type_ of the node is set as _link node_
     3. The _linked content start position_ of the node is set to
        ( [current-position] + 1 )

 2. Set [consumed-character-count] to 1

If the character at the [current-position] is a `]` character, it might
be the start of a _closing link tag_. In this case, the following is
done:

 1. If the [topmost node of type] _link node_ is not
    [_null_](#topmost-node-of-type-is-null), and if the
    [remaining-character-sequence] matches the regular expression
    pattern `/^\]\s*\[(([^\\\[\]]|\\.)+)\]/` (Example: `] [ref id]`),
    then the following is done:

     1. The matching substring for the whole of the pattern is
        identified as a **closing link tag**.

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
        [de-escaped] and then [attribute-value-escaped].

        If the [link reference association map] does not contain an
        entry for _reference id string_, then the output shall have the
        _enclosed content_ without being part of a link,
        enclosed within the text forming the _opening link tag_ and the
        text forming the _closing link tag_. For HTML output, the text
        forming the _closing link tag_ should be [de-escaped] and then
        [html-text-escaped] before being output.

     7. The [top node] is popped off

     8. All nodes with _node type_ equal to _link node_ are removed from
        the [stack of potential opening span tags] and interpreted as
        _text fragments_

     9. Set [consumed-character-count] to the number of characters in
        the _closing link tag_

 2. If the [topmost node of type] _link node_ is not
    [_null_](#topmost-node-of-type-is-null), and if both the following
    conditions are satisfied:

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

         2. Title and closing paranthesis:  
            `/^\s*("(([^"\\]|\\.)*)"|'(([^'\\]|\\.)*)')\s*\)/`

            Examples:  
            `"Title")`  
            `'Title')`  
            `"A (nice) \"title\" for the 'link'")`

            If this is the matching pattern, the matching substring for
            the first (i.e. outer) parenthesized subexpression in the
            pattern is called the _attributes-string_. The
            _attributes-string_ will be a [quoted string], and the
            [enclosed string] of the [quoted string] is called the
            _unprocessed title string_. Any [line break] characters in
            the _unprocessed title string_ are removed, and the
            resultant string is called the _title string_.

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
        **closing link tag**

     2. If the [topmost node of type] _link node_ is not already the
        [top node], all nodes above it are popped off and interpreted as
        _text fragments_

     3. The [top node] \(which should have its _node type_ equal to _link
        node_\) is interpreted as an _opening span tag_, or more
        specifically, as an **opening link tag**

     4. The _closing link tag_ is said to correspond to the _opening
        link tag_, and any _span tags_ or _text fragments_ occuring
        between the _opening link tag_ and the _closing link tag_ are
        considered to form the _enclosed content_ of the link

     5. The _link url string_ shall be used as the link url for the
        link.  If the _title string_ is not _null_, the _title string_
        shall be used as the title for the link. The output shall have
        the _enclosed content_ linked to the link url and link title.
        For HTML output, the link title should be [de-escaped] and then
        [attribute-value-escaped].

     6. The [top node] is popped off

     7. All nodes with _node type_ equal to _link node_ are removed from
        the [stack of potential opening span tags] and interpreted as
        _text fragments_

     8. Set [consumed-character-count] to _close-link-tag-length_

 3. If neither of the above conditions are satisfied, and if the
    [topmost node of type] _link node_ is not
    [_null_](#topmost-node-of-type-is-null), and if the
    character at the [current-position] is a `]` character, then the
    following is done:

     1. Let _empty-ref-pattern_ be the regular expression pattern
        `/^(\]\s*\[\s*\])/` (Example: `][]`)

        If the [remaining-character-sequence] matches the
        _empty-ref-pattern_, then the length of the matching substring
        for the whole pattern is said to be the _close-link-tag-length_.

        If the [remaining-character-sequence] does not match the
        _empty-ref-pattern_, then the _close-link-tag-length_ is said to
        be 1.

        The first _close-link-tag-length_ characters of the
        [remaining-character-sequence] are collectively identified as a
        **closing link tag**

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
        [de-escaped] and then [attribute-value-escaped].

        If the [link reference association map] does not contain an
        entry for _reference id string_, then the output shall have the
        _enclosed content_ without being part of a link,
        enclosed within the text forming the _opening link tag_ and the
        text forming the _closing link tag_.

     6. The [top node] is popped off

     7. All nodes with _node type_ equal to _link node_ are removed from
        the [stack of potential opening span tags]

     8. Set [consumed-character-count] to the number of characters in
        the _closing link tag_

 4. If the [topmost node of type] _link node_ is
    [_null_](#topmost-node-of-type-is-null), and if the character at the
    [current-position] is a `]` character, then the `]` at the
    [current-position] is interpreted as a _text fragment_, and
    [consumed-character-count] is set to 1

<h3 id="procedure-for-identifying-emphasis-tags">
Procedure for identifying emphasis tags</h3>

[Procedure for identifying emphasis tags]: #procedure-for-identifying-emphasis-tags
[procedure for identifying emphasis tags]: #procedure-for-identifying-emphasis-tags

This procedure assumes that the character at the [current-position] is
either an unescaped `*` character or an unescaped `_` character.

<span id="emphasis-fringe-rank">We define **emphasis-fringe-rank** of a
[character] based on the 'General\_Category' unicode property as
follows:</span>

 1. If the 'General\_Category' unicode property of the character is one
    of the following: Zs, Zl, Zp, Cc or Cf, then the _emphasis fringe
    rank_ of the character is said to be 0.

    For example, [space] and [line break] characters have an _emphasis
    fringe rank_ of 0.

 2. If the 'General\_Category' unicode property of the character is one
    of the following: Pc, Pd, Ps, Pe, Pi, Pf or Po, then the _emphasis
    fringe rank_ of the character is said to be 1.

    For example, the following characters have an _emphasis fringe rank_
    of 1: `,`, `.`, `(`, `)`, `+`

 3. If the 'General\_Category' unicode property of the character is not
    one of the following: Zs, Zl, Zp, Cc, Cf, Pc, Pd, Ps, Pe, Pi, Pf or
    Po, then the _emphasis fringe rank_ of the character is said to be 2.

    For example, alphanumeric characters have an _emphasis fringe rank_
    of 2.

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

<span id="right-fringe-rank">
In case the match is with the first regular expression pattern given
above, then the _right-fringe-rank_ of the [emphasis indicator string] is
said to be 0. In case the match is with the second regular expression
pattern given above, then the _right-fringe-rank_ of the [emphasis
indicator string] is the [emphasis-fringe-rank] of the single character
in the matching substring for the second parenthesized subexpression in
the pattern.</span>

<span id="left-fringe-rank">
If the [current-position] is equal to 0, the _left-fringe-rank_ of the
[emphasis indicator string] is said to be 0. If the [current-position]
is greater than 1, then the _left-fringe-rank_ of the [emphasis indicator
string] is the [emphasis-fringe-rank] of the character at the previous
position (i.e. at [current-position] minus 1).</span>

<span id="flanking">
For a given [emphasis indicator string], if its [left-fringe-rank] is
lesser than its [right-fringe-rank], the [emphasis indicator string] is
said to be _left-flanking_. On the other hand, if its [right-fringe-rank]
is lesser than its [left-fringe-rank], the [emphasis indicator string] is
said to be _right-flanking_. If the [left-fringe-rank] of an [emphasis
indicator string] is equal to its [right-fringe-rank], the [emphasis
indicator string] is said to be _non-flanking_.</span>

Consider the following example:

    ***Shaken*, ** not _stirred_**.

There are 5 _emphasis indicator strings_ in the above example. The
_left-fringe-rank_ and _right-fringe-rank_ of each is given below:

<table>
<tr>
    <th>#</th>
    <th><em>emphasis indicator string</em></th>
    <th><em>left fringe rank</em></th>
    <th><em>right fringe rank</em></th>
    <th>flankingness</th>
</tr>
<tr>
    <td>1</td>
    <td><code>***</code></td>
    <td>0</td>
    <td>2</td>
    <td><em>left-flanking</em></td>
</tr>
<tr>
    <td>2</td>
    <td><code>*</code></td>
    <td>2</td>
    <td>1</td>
    <td><em>right-flanking</em></td>
</tr>
<tr>
    <td>3</td>
    <td><code>**</code></td>
    <td>0</td>
    <td>0</td>
    <td><em>non-flanking</em></td>
</tr>
<tr>
    <td>4</td>
    <td><code>_</code></td>
    <td>0</td>
    <td>2</td>
    <td><em>left-flanking</em></td>
</tr>
<tr>
    <td>5</td>
    <td><code>_**</code></td>
    <td>2</td>
    <td>1</td>
    <td><em>right-flanking</em></td>
</tr>
</table>

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

[emphasis-fringe-rank]: #emphasis-fringe-rank
[left-fringe-rank]: #left-fringe-rank
[right-fringe-rank]: #right-fringe-rank
[emphasis indicator string]: #emphasis-indicator-string>
[left-flanking]: #flanking
[right-flanking]: #flanking
[non-flanking]: #flanking
[emphasis tag string]: #emphasis-tag-string
[emphasis tag strings]: #emphasis-tag-string
[constituent character]: #constituent-character

If the [emphasis indicator string] is [non-flanking], the the [emphasis
indicator string] is interpreted as a _text fragment_. The
[consumed-character-count] is set to the length of the [emphasis
indicator string].

If the [emphasis indicator string] is [left-flanking], then the
[emphasis tag strings] in the [emphasis indicator string] can
potentially become _opening emphasis tags_. In this case, the following
shall be done:

 1. For each [emphasis tag string] in the [emphasis indicator string]
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

 2. Set [consumed-character-count] to the length of the [emphasis
    indicator string]

If the [emphasis indicator string] is [right-flanking], then the
[emphasis tag strings] in the [emphasis indicator string] can
potentially be interpreted as _closing emphasis tags_. In this case, the
following shall be done:

 1. Set _current-tag-string_ to the first [emphasis tag string] in
    the [emphasis indicator string]

 2. <span id="emphasis-proc-step-2">If the [constituent character] of
    the _current-tag-string_ is `*`, then the _matching emphasis node_
    is the [topmost node of type] _asterisk emphasis node_; if the
    [constituent character] of the _current-tag-string_ is `_`, then the
    _matching emphasis node_ is the [topmost node of type] _underscore
    emphasis node_</span>

 3. If _matching emphasis node_ is
    [_null_](#topmost-node-of-type-is-null), then the _emphasis tag
    string_ is interpreted as a _text fragment_

 4. If the _matching emphasis node_ is not
    [_null_](#topmost-node-of-type-is-null), and if it is not already
    the [top node], then all nodes above it are popped off and
    interpreted as _text fragments_

 5. If the _matching emphasis node_ is not
    [_null_](#topmost-node-of-type-is-null), then invoke the [procedure
    for matching emphasis tag strings]. The _current-tag-string_ and/or
    the [top node] can get modified within that procedure.

 6. If the _current-tag-string_ is empty, and if there are any more
    unprocessed [emphasis tag strings] in the [emphasis indicator
    string], set the _current-tag-string_ to the next
    [emphasis tag string]

 7. If the _current-tag-string_ is not empty, go to
    [Step 2](#emphasis-proc-step-2)

 8. Set [consumed-character-count] to the length of the [emphasis
    indicator string]

<h4 id="matching-opening-and-closing-emphasis">
Procedure for matching emphasis tag strings</h4>

[Procedure for matching emphasis tag strings]: #procedure-for-matching-emphasis-tag-strings
[procedure for matching emphasis tag strings]: #procedure-for-matching-emphasis-tag-strings

This procedure describes how the _current-tag-string_ is to be matched
with the _matching emphasis node_ at the top of the [stack of potential
opening span tags].

In this procedure, the [top node] is assumed to be of a _node type_ that
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

     1. The [top node] is interpreted as an **opening emphasis tag**

     2. The _current-tag-string_ is interpreted as **closing emphasis tag**

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
        tag string_ are interpreted as an **opening emphasis tag**.

     2. The whole of the _current-tag-string_ is interpreted a **closing
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

<h3 id="procedure-for-identifying-code-span-tags">
Procedure for identifying code-span tags</h3>

[Procedure for identifying code-span tags]: #procedure-for-identifying-code-span-tags
[procedure for identifying code-span tags]: #procedure-for-identifying-code-span-tags

This procedure assumes that the character at the [current-position] is
an unescaped `` ` `` character.

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

 3. <span id="code-proc-step-3">If the _residual-code-span-sequence_
    matches one of the following regular expression patterns:</span>

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
    characters, go to [Step 3](#code-proc-step-3)

 5. If a _closing-code-string_ has not yet been identified, the
    _opening-backticks-string_ is identified as a _text fragment_, and
    [consumed-character-count] is set to the length of the
    _opening-backticks-string_

 6. If a _closing-code-string_ has been identified, the following is
    done:

     1. Let _code-span-length_ be equal to
        ( ( _opening-backticks-count_ * 2 ) + _code-content-length_ )
     2. The first _code-span-length_ characters of the
        [remaining-character-sequence] is identified as a **code span
        tag**
     3. Among the characters that form the _code span tag_, the first
        _opening-backticks-count_ characters and the last
        _opening-backticks-count_ characters are considered to be
        markup. The middle _code-content-length_ characters constitute
        the _unprocessed-code-content-string_. The
        _unprocessed-code-content-string_ is [trimmed] to form
        the content of the code span. For HTML output, the content of
        the code-span should be [html-text-escaped].
     4.  Set [consumed-character-count] to _code-span-length_

<h3 id="procedure-for-identifying-image-tags">Procedure for identifying image tags</h3>

[Procedure for identifying image tags]: #procedure-for-identifying-image-tags
[procedure for identifying image tags]: #procedure-for-identifying-image-tags

This procedure assumes that the character at the [current-position] is
an unescaped `!` character, and that the immediate next character is a
`[` character.

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

 1. If the [residual-image-sequence] matches the regular expression
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
        [remaining-character-sequence] are collectively identified as an
        **image tag**.

     3. The _reference id string_ shall be used to look up the actual
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
        output, this text should be [html-text-escaped].

     4. Set [consumed-character-count] to _image-ref-tag-length_

 2. If the [residual-image-sequence] matches the regular expression
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

         2. Title and closing paranthesis:  
            `/^\s*("(([^"\\]|\\.)*)"|'(([^'\\]|\\.)*)')\s*\)/`

            Examples:  
            `"Title")`  
            `'Title')`  
            `"A (nice) \"title\" for the 'image'")`

            If this is the matching pattern, the matching substring for
            the first (i.e. outer) parenthesized subexpression in the
            pattern is called the _attributes-string_. The
            _attributes-string_ will be a [quoted string], and the
            [enclosed string] of the [quoted string] is called the
            _unprocessed title string_. Any [line break] characters in
            the _unprocessed title string_ are removed, and the
            resultant string is called the _title string_.

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
        _image-attributes-pattern-match-length_. The first
        _image-src-tag-length_ characters of the
        [remaining-character-sequence] are collectively identified as an
        **image tag**.

     2. The _image url string_ shall be used as the source of the image.
        If _title string_ is not null, the _title string_ shall be used
        as the title of the image. The [image-alt-text-string] shall be
        used as the alternate text for the image. For HTML output, the
        title of the image and the alternate text for the image should
        be [attribute-value-escaped].

     3. Set [consumed-character-count] to _image-src-tag-length_

 3. If neither of the above conditions are satisfied, then the following
    is done:

     1. Let _empty-ref-pattern_ be the regular expression pattern
        `/^(\]\s*\[\s*\])/` (Example: `][]`)

        If the remaining-character-sequence matches the
        _empty-ref-pattern_, then the length of the matching substring
        for the whole pattern is said to be the
        _image-ref-close-sequence-length_.

        If the remaining-character-sequence does not match the
        _empty-ref-pattern_, then the _image-ref-close-sequence-length_
        is said to be 1.

     2. Let _image-ref-tag-length_ be equal to the sum of
        [alt-text-pattern-match-length] and
        _image-ref-close-sequence-length_. The first
        _image-ref-tag-length_ characters of the
        [remaining-character-sequence] are collectively identified as an
        **image tag**.

     3. Let _reference id string_ be the string obtained on simplifying
        the [image-alt-text-string].

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
        output, this text should be [html-text-escaped].

     4. Set [consumed-character-count] to _image-ref-tag-length_


<h3 id="procedure-for-detecting-automatic-links">Procedure for detecting automatic links</h3>

[Procedure for detecting automatic links]: #procedure-for-detecting-automatic-links
[procedure for detecting automatic links]: #procedure-for-detecting-automatic-links

<span id="word-separator">We define a **word-separator** [character] to
be a unicode code point whose 'General\_Category' unicode property has
one of the following values:</span>

 1. One of: Zs, Zl, Zp (i.e. a 'Separator')

    (or)

 2. One of: Pc, Pd, Ps, Pe, Pi, Pf, Po (i.e. a 'Punctuation')

    (or)

 3. One of: Cc, Cf

For example, the [space] character, the [line break] character, `.`,
`,`, `(`, `)` are all _word-separator_ characters.

[word-separator]: #word-separator

If any one of the following conditions are satisfied:

 1. The character at the [current-position] is a `<` character

 2. The [current-position] is equal to 1

 3. The [current-position] is greater than 1, and the character at
    ([current-position] - 1) is a [word-separator] character

then the [current-position] is said to be a
_potential-auto-link-start-position_.

If the [current-position] is a _potential-auto-link-start-position_,
then the following is done:

 1. If the [remaining-character-sequence] matches one of the following
    regular expression patterns (matching shall be case insensitive):

     1. URL within angle brackets:
        `/<([a-z0-9\+\.\-]+:\/\/[^<>\s]+)>/`

        Example: `<http://example.net>`

     2. Mailto URL within angle brackets:
        `/<(mailto:[^<>\s]+)>/`

        Example: `<mailto:someone@example.net?subject=Hi+there>`


    then the following is done:

    <!-- For some reason, Redcarpet requires a comment here to correctly
    display the following list -->

     1. The matching substring for the whole of the matching pattern is
        identified as an **auto-link tag**
     2. The matching substring for the first parenthesized subexpression in
        the matching pattern is called the _auto-link url_
     3. The output shall have a link with the link url set as _auto-link
        url_ and the link text content also set as _auto-link url_
     4. The [consumed-character-count] is set to the length of the
        _auto-link tag_

 2. If the [remaining-character-sequence] matches the regular expression
    pattern `/<([^\/\?#@\s]+@[^\/\?#@\s\.]+\.[^\/\?#@\s]+)>/`
    (Example: `<someone@example.net>`), then the following is done:

     1. The matching substring for the whole of the matching pattern is
        identified as an **auto-link tag**
     2. The matching substring for the first parenthesized subexpression in
        the matching pattern is called the _auto-link email_. Let the
        string formed by concatenating the string `mailto:` with the
        _auto-link email_ be called as _auto-link email url_
     3. The output shall have a link with the link url set as _auto-link
        email url_ and the link text content set as _auto-link email_
     4. The [consumed-character-count] is set to the length of the
        _auto-link tag_

 3. If the [remaining-character-sequence] matches one of the following
    regular expression patterns (matching shall be greedy and case
    insensitive):

     1. URL without angle brackets:
        `/([a-z0-9\+\.\-]+:\/\/)[^<>\s]+/`

        Example: `http://example.net`

     3. Mailto URL without angle brackets:
        `/(mailto:)[^<>\s]+/`

        Example: `mailto:someone@example.net`

    then the following is done:

    <!-- For some reason, Redcarpet requires a comment here to correctly
    display the following list -->

     1. The matching substring for the whole of the matching pattern is
        called the _unprocessed auto-link tag_
     2. The matching substring for the first and only parenthesized
        subexpression in the pattern is called the _auto-link scheme
        string_. The number of characters in the _auto-link scheme
        string_ is called the _auto-link scheme string length_.
     3. The _unprocessed auto-link tag_ shall be processed to give the
        _auto-link tag candidate_. The processing to be done is to
        remove any trailing [word-separator] characters, such that the
        last character of the _auto-link tag candidate_ is not a
        [word-separator] character.
     4. If the length of the _auto-link tag candidate_ is greater than
        the _auto-link scheme string length_, then the _auto-link tag
        candidate_ is identified as an **auto-link tag**, and the
        following is done:
         1. The output shall have a link with the link url set as
            _auto-link tag candidate_ and the link text content also set
            as _auto-link tag candidate_
         2. The [consumed-character-count] is set to the length of the
            _auto-link tag candidate_
     5. If the length of the _auto-link tag candidate_ is lesser than or
        equal to the _auto-link scheme string length_, then the
        _auto-link scheme string_ is identified as a _text fragment_,
        and the [consumed-character-count] is set to _auto-link scheme
        string length_


<h3 id="procedure-for-identifying-html-tags">Procedure for identifying HTML tags</h3>

[Procedure for identifying HTML tags]: #procedure-for-identifying-html-tags
[procedure for identifying HTML tags]: #procedure-for-identifying-html-tags

This procedure assumes that the character at the [current-position] is
an unescaped `<` character.

<span id="verbatim-html-starter-tag-name"> We define a
**verbatim-html-starter-tag-name** to be one of the following HTML tag
names:</span> `address`, `article`, `aside`, `blockquote`, `details`,
`dialog`, `div`, `dl`, `fieldset`, `figure`, `footer`, `form`, `header`,
`main`, `nav`, `ol`, `section`, `table` or `ul`.

<span id="verbatim-html-container-tag-name"> We define a
**verbatim-html-container-tag-name** to be one of the following HTML tag
names:</span> `pre`, `script` or `style`.

[verbatim-html-starter-tag-name]: #verbatim-html-starter-tag-name
[verbatim-html-container-tag-name]: #verbatim-html-container-tag-name

Let _html-tag-detection-sequence_ be the [remaining-character-sequence].

To identify _span tags_ related to inline HTML we need to employ the use
of a HTML parser. We supply characters in the
_html-tag-detection-sequence_ to the HTML parser, one character at a
time, till one of the following happens:

 1. The HTML parser detects a complete self-closing HTML tag (this also
    includes tags empty by definition in HTML4, like `<br>` and `<img
    src="picture.jpg">`, for example)

    If this happens first, the following is done:

     1. The text that represents the self-closing HTML tag is identified
        as a **self-closing HTML tag**

     2. For HTML output, the text that represents the self-closing HTML
        tag shall be included in the output verbatim

     3. Set [consumed-character-count] to the length of the
        _self-closing HTML tag_

     4. If the HTML tag name of the self-closing HTML tag is either a
        [verbatim-html-starter-tag-name] or a
        [verbatim-html-container-tag-name], then set
        [is-verbatim-html-mode] to _true_

 2. The HTML parser detects a complete opening HTML tag.

    If this happens first, the following is done:

     1. The text that represents the opening HTML tag is identified as
        an **opening HTML tag**

     2. A new node is pushed onto the [stack of potential opening span
        tags] with the following [properties][stack-node-properties]:

         1. The _tag string_ of the node is set to the text that
            represents the opening HTML tag
         2. The _node type_ of the node is set as _raw html node_
         3. The _html tag name_ of the node is set to the HTML tag name
            of the opening HTML tag that was just identified

     3. For HTML output, the text that represents the opening HTML tag
        shall be included in the output verbatim.

     4. Set [consumed-character-count] to the length of the
        _opening HTML tag_

     5. If the HTML tag name of the opening HTML tag is either a
        [verbatim-html-starter-tag-name] or a
        [verbatim-html-container-tag-name], then set
        [is-verbatim-html-mode] to _true_

 3. The HTML parser detects a complete closing HTML tag.

    If this happens first, the following is done:

     1. The text that represents the closing HTML tag is identified as a
        **closing HTML tag**

     2. Let _currently open html node_ be the [topmost node of type]
        _raw html node_

     3. If the _currently open html node_ is not
        [_null_](#topmost-node-of-type-is-null), and the _html tag name_
        of the _currently open html node_ is the same as the HTML tag
        name of the closing HTML tag that was just identified, the [top
        node] shall be popped off

     4. If the _currently open html node_ is
        [_null_](#topmost-node-of-type-is-null), or if  _html tag name_
        of the _currently open html node_ is not the same as the HTML
        tag name of the closing HTML tag that was just identified, then
        all nodes in the [stack of potential opening span tags] whose
        _node type_ is not equal to _raw html node_ shall be removed
        from the stack and interpreted as _text fragments_

     5. For HTML output, the text that represents the closing HTML tag
        shall be included in the output verbatim.

     6. Set [consumed-character-count] to the length of the
        _closing HTML tag_

     7. If the HTML tag name of the closing HTML tag is either a
        [verbatim-html-starter-tag-name] or a
        [verbatim-html-container-tag-name], then set
        [is-verbatim-html-mode] to _true_

 4. The HTML parser detects a complete HTML comment.

    If this happens first, the text that represents the HTML comment is
    identified as a _span tag candidate_, and interpreted as a **comment
    HTML tag**.

    For HTML output, the text that represents the comment HTML tag shall
    be included in the output verbatim.

    [consumed-character-count] is set to the length of the _comment HTML
    tag_.

 5. The HTML parser detects HTML text, or the HTML parser detects an
    error, or the _html-tag-detection-sequence_ has no more characters
    to supply to the HTML parser.

    If this happens first, the `<` at the [current-position] is
    identified as a _text fragment_, and [consumed-character-count] is
    set to 1.

<h2 id="additional-processing">Additional processing</h2>

[Additional processing]: #additional-processing

Some additional processing is required for certain parts before they can
be written to the output.

<h3 id="de-escaping">De-escaping</h3>

[De-escaping]: #de-escaping
[de-escaped]: #de-escaping

Escaping backslashes in the input should not be part of the output.

<span id="punctuation">We define a **punctuation** [character] to
be a unicode code point whose 'General\_Category' unicode property has
one of the following values: Pc, Pd, Ps, Pe, Pi, Pf, Po.</span>

[punctuation]: #punctuation

To de-escape a [string], every `\` (backslash) character in the string
that is used for [escaping] a [punctuation] character, shall be removed.

For example, for the string `With \(esca\ped\) \\brackets`, the
de-escaped string will be `With (esca\ped) \brackets`.

<h3 id="processing-text-fragments">Processing text fragments</h3>

[Processing text fragments]: #processing-text-fragments
[processing text fragments]: #processing-text-fragments

The _text fragments_ identified in the [procedure for identifying span
tags] are subject to the following processing:

 1. The _text fragments_ that occur adjacent to one another in the
    [input character sequence] are collated to form a single _collated
    text fragment_
 2. Every _collated text fragment_ is processed to produce a _processed
    text fragment_. The following processing is done:

     1. **Removing escaping backslashes:** The _collated text fragment_
        shall be [de-escaped].

     2. **Introducing hard-breaks:** Every sequence of two [space]
        characters followed by a [line break] character shall be
        replaced by a hard line break as appropriate for the output
        format. For HTML output format, a `<br />` element is used to
        indicate a hard line break.

        For example, for the following _collated text fragment_:

            There are two spaces at the end of this line  
            So we introduce a hard break there

        the corresponding _processed text fragment_ for HTML output will
        be:

            There are two spaces at the end of this line<br />
            So we introduce a hard break there

     3. **HTML escaping:** For HTML output, the _collated text fragment_
        shall be [html-text-escaped].

 3. The _processed text fragment_ is output


<h3 id="processing-for-html-output">Processing for HTML output</h3>

[Processing for HTML output]: #processing-for-html-output

For HTML output, the text that shall be output as part of text content
of a HTML element, or the text that shall be output as part of a HTML
tag's attribute value, needs to be escaped as described in this section.

<h4 id="html-text-escaping">HTML text escaping</h4>

[HTML text escaping]: #html-text-escaping
[html-text-escaped]: #html-text-escaping

**HTML text escaping** involves the following replacements:

 1. Replace the `<` character with `&lt;`
 2. Replace the `>` character with `&gt;`
 3. Replace the `&` character with `&amp;`

<h4 id="attribute-value-escaping">Attribute value escaping</h4>

[Attribute value escaping]: #attribute-value-escaping
[attribute-value-escaped]: #attribute-value-escaping

**Attribute value escaping** involves the following replacements:

 1. Replace the `<` character with `&lt;`
 2. Replace the `>` character with `&gt;`
 3. Replace the `&` character with `&amp;`
 4. Replace the `"` character with `&quot;`
 4. Replace the `'` character with `&#39;`

It is recommended that the `"` character be used as the enclosing
quote character for attribute values.

Note that the above rules for attribute values only apply for the
attributes of HTML elements output as a result of a non-raw-HTML vfmd
construct. While outputting
[verbatim HTML](#procedure-for-identifying-html-tags), the attribute
values should be written exactly as present in the source vfmd document.

<h2 id="extending-the-syntax">Extending the syntax</h2>

[Extending the syntax]: #extending-the-syntax

An implementation can extend the [core syntax] to support additional
syntax elements. The additional syntax elements can involve [block-level
extensions], or [span-level extensions], or both.

[core syntax]: introduction.md#core-syntax

For example, to support [GitHub-style fenced code blocks], an
implementation would need to add a [block-level extension]; to support
[GitHub-style strikethroughs], an implementation would need to add a
[span-level extension]; to support [MultiMarkdown-style footnotes], an
implementation would need to add both a [block-level extension] \(for
handling footnote definitions\) and a [span-level extension] \(for
handling footnote references\).

[GitHub-style fenced code blocks]: https://help.github.com/articles/github-flavored-markdown#fenced-code-blocks
[GitHub-style strikethroughs]: https://help.github.com/articles/github-flavored-markdown#strikethrough
[MultiMarkdown-style footnotes]: http://fletcher.github.io/peg-multimarkdown/#footnotes

