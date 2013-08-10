# vfmd userguide

This document describes the [vfmd] syntax in simple terms, and
is intended to be of help for people writing in vfmd.

The vfmd syntax is pretty much the same as the [original Markdown
syntax], with a few minor changes and added clarifications.

vfmd enables you to represent rich-text markup in plain-text files. It
supports block-level markup like lists, blockquotes and code-blocks, and
also supports span-level markup like emphasis, links and inline-code.

[vfmd]: introduction.md
[original Markdown syntax]: http://daringfireball.net/projects/markdown/syntax

## Encoding

The input text should be in UTF-8 encoding. If you are writing your vfmd
document in English, it is most likely in UTF-8. If you are writing in
another language, please make sure that the document is in UTF-8
encoding (for example, if you are using a text editor to write the
document, ensure that it saves the document in UTF-8 encoding).

## Block-level elements

### Headers

vfmd supports both [setext]-style and [atx]-style headers.

[setext]: http://docutils.sourceforge.net/mirror/setext.html
[atx]: http://www.aaronsw.com/2002/atx/

#### setext-style headers

To create a setext-style header, follow the header text with a line that
"underlines" the text with equal signs (`=`) or dashes (`-`).

For a first-level header, the "underline" line should be sequence of one
or more `=` characters. For a second-level header, it should be a
sequence of one or more `-` characters. There can be no other characters
in the "underline" line (trailing spaces are okay).

For example:

    This is a first-level header
    ============================

    This is a second-level header
    -----------------------------

    This is a first-level header
    ===

    This is a second-level header
    ---

Please note that in case the header is in the immediate next line of a
paragraph, it will be considered as a part of the paragraph, and shall
not be identified as a header.

    This is a paragraph. If you don't inculde a blank line
    between this paragraph of text and the title of the next
    section, the title will not be recognized as a title.
    This will not be recognized as a section title
    ----------------------------------------------

    If you do include a blank line between the paragraph and
    the title, it all works out correctly.

    This is a section title
    -----------------------

#### atx-style headers

atx-style headers start with one or more `#` characters. The first-level
header starts with a single `#`, second-level headers with `##`,
third-level headers with `###`, and so on, till 6 `#` characters for a
sixth-level header.

For example:

    # Title (first-level header)

    ## Section (second-level header)

    ### Subtitle (third-level header)

    ###### Sixth-level header

    ######### Still a sixth-level header

You may optionally "close" the header with any number of additional `#`
characters. However, the header level is determined only by counting the
"opening" `#` characters.

For example:

    ## Second-level header ##

    ### Third-level header ######

    ###### Sixth-level header #

Please note that in case the header is in the immediate next line of a
paragraph, it will be considered as a part of the paragraph, and shall
not be identified as a header.

    This is a paragraph. If you don't inculde a blank line
    between this paragraph of text and the title of the next
    section, the title will not be recognized as a title.
    # This will not be recognized as a section title

    If you do include a blank line between the paragraph and
    the title, it all works out correctly.

    # This is a section title

### Code blocks

Code blocks can be used to quote text verbatim. For example, it can be
used to quote source code. Every line of the code block should be
indented by 4 space characters.

No Markdown syntax is processed within a code block. In addition, for
HTML output, ampersands (`&`) and angle brackets (`<` and `>`) within a
code block are automatically converted into HTML entities, so that the
text appears as specified when the output HTML in viewed in a browser.

For example:

    Consider the following snippet that mixes Markdown with HTML5:

        ## Introduction

        This is <del>normal text</del> an intro.

becomes, in HTML output:

    <p>Consider the following snippet that mixes Markdown with HTML5:</p>

    <pre><code>## Introduction

    This is &lt;del&gt;normal text&lt;/del&gt; an intro.
    </pre></code>

### Blockquotes

Blockquoting is done by starting a line with `>` characters, like it's
done in email. It looks best if you hard wrap the text and start every
line with a `>`, like this:

    > 'Very true,' said the Duchess: 'flamingoes and mustard both
    > bite.  And the moral of that is - "Birds of a feather flock
    > together."'
    >
    > 'Only mustard isn't a bird,' Alice remarked.
    >
    > 'Right, as usual,' said the Duchess: 'what a clear way you
    > have of putting things!'

But you are allowed to skip the leading `>` for subsequent lines of a
hard-wrapped paragraph, as long as the first line of the paragraph
starts with a `>`. If there's only one blank line between multiple
blockquoted paragraphs, they form a single blockquote, like this:

    > 'Very true,' said the Duchess: 'flamingoes and mustard both
    bite.  And the moral of that is - "Birds of a feather flock
    together."'
    
    > 'Only mustard isn't a bird,' Alice remarked.
    
    > 'Right, as usual,' said the Duchess: 'what a clear way you
    have of putting things!'

Blockquotes can contain other elements, like headers, code blocks and
nested blockquotes.

For example:

    > ## Section header
    >
    > This is a paragraph inside a blockquote.
    >
    > > This is a nested blockquote.
    > > The second line of the nested blockquote.
    > >
    > > > This is the third level of nesting.
    >
    > Some code that gives the ultimate answer of
    > _Life, the Universe and Everything_:
    >
    >     int main() {
    >         return 42;
    >     }

### Horizontal rule

A horizontal rule tag (`<hr/>`) can be created by placing three or more
hyphens (`-`), or three or more underscores (`_`) or three or more
asterisks (`*`) on a line by themselves. The hyphens (or the
underscores, or the asterisks) forming the horizontal rule can
optionally be interspersed with spaces.

Each of the following lines results in a horizontal rule:

    ***

    *****

    * * * *

    -------------
    _   _   _   _   _   _   _

    -- --- --

But the following lines don't result in a horizontal rule:

    **

    --

    *-*-*

    _-_-_-_


## Mixing HTML with vfmd

If you are capable of authoring HTML documents, you can use snippets of
HTML in your vfmd document to augument the vfmd feature set. That said,
if you would like to convert a vfmd source document to any output format
other than HTML, the support for using HTML tags in the vfmd source
document is implementation-dependant.

vfmd handles different HTML elements differently. In vfmd, HTML elements
are seen as belonging to one of the following four groups:

 1. **Phrasing HTML elements** are HTML elements that belong to the
    [phrasing content] category in [HTML5]. The elements in this group
    are: `a`, `abbr`, `area`, `b`, `bdi`, `bdo`, `br`, `button`,
    `canvas`, `cite`, `code`, `data`, `datalist`, `del`, `dfn`, `em`,
    `embed`, `i`, `iframe`, `img`, `input`, `ins`, `kbd`, `keygen`,
    `label`, `map`, `mark`, `meter`, `noscript`, `object`, `output`,
    `progress`, `q`, `ruby`, `s`, `samp`, `select`, `small`, `span`,
    `strong`, `sub`, `sup`, `textarea`, `time`, `u`, `var` and `wbr`.
 2. **Verbatim HTML starter elements** are HTML elements that are [flow
    content], but not [phrasing content], and can in turn
    [contain][content models] [flow content]. The elements in this group
    are: `address`, `article`, `aside`, `blockquote`, `details`,
    `dialog`, `div`, `dl`, `fieldset`, `figure`, `footer`, `form`,
    `header`, `main`, `nav`, `ol`, `section`, `table` and `ul`.
 3. **Verbatim HTML container elements** are HTML elements within which
    vfmd syntax should never be recognized. `pre`, `style` and `script`
    elements fall in this group.
 4. **Other HTML elements** are HTML elements that don't belong to any
    of the above groups.

[HTML5]: http://www.w3.org/TR/html5/ "HTML5 Specification"
[content models]: http://www.w3.org/TR/html5/dom.html#content-models
[flow content]: http://www.w3.org/TR/html5/dom.html#flow-content-1
[phrasing content]: http://www.w3.org/TR/html5/dom.html#phrasing-content-1

### Using vfmd along with HTML markup

_Phrasing HTML elements_ can be freely intermingled with vfmd text. Such
HTML elements can contain, and be contained in, vfmd markup.

For example:

    According to the <u>_Special_ Theory of Relativity</u>,
    **E** <em>(energy)</em> = **mc<sup>2</sup>**

becomes, in HTML output:

    <p>According to the <u><em>Special</em> Theory of Relativity</u>,
    <strong>E</strong> <em>(energy)</em> = <strong>mc<sup>2</sup></strong></p>

_Other HTML elements_ can contain vfmd text, but cannot be contained
within vfmd markup. Moreover, vfmd paragraphs containing any of the
_other HTML elements_ are not wrapped in `p` tags.

The following example shows how vfmd markup can be used within `td`
elements of a HTML table:

    <td>Apply some _stress_</td>
    <td>Make it sound **important**</td>

The HTML output for the above example shall not have any `p` tags:

    <td>Apply some <em>stress</em></td>
    <td>Make it sound <strong>important</strong></td>

Also, if the vfmd paragraph contains any mismatched or misnested HTML
tags (i.e. opening tags without correctly-placed closing tags or vice
versa), vfmd will not wrap the paragraph content in `p` tags, because
doing so might result in invalid HTML output.

### Verbatim HTML

Anything within a _verbatim HTML container element_ (i.e. a `pre`,
`script` or `style` element) is preserved as-is in the HTML output.

Moreover, any tag (opening, closing or self-closing tag) of a _verbatim
HTML starter element_ marks the start of verbatim HTML, and the
verbatim-HTML-mode stays on till the next blank line.

So, if you'd like to use vfmd syntax within a _verbatim HTML starter
element_ (like `div` or `table`), you can use one or more blank lines to
separate the parts that need to be output verbatim, from the parts in
which vfmd syntax needs to be processed.

For example, for the input:

    <table>
      <tr>
        <th>Single *asterisk* or _underscore_</th>
        <th>Double **asterisks** or __underscores__</th>
      </tr>
      <tr>

    <td>Apply some _stress_</td>
    <td>Make it sound **important**</td>

      </tr>
    </table>

The corresponding HTML output shall be:

    <table>
      <tr>
        <th>Single *asterisk* or _underscore_</th>
        <th>Double **asterisks** or __underscores__</th>
      </tr>
      <tr>

    <td>Apply some <em>stress</em></td>
    <td>Make it sound <strong>important</strong></td>

      </tr>
    </table>

However, please make sure that the starting line of each snippet of HTML
is not indented by more than 3 spaces (if it's indented by 4 or more
spaces, it would become a code block).

On the contrary, if you want the whole block of HTML reproduced verbatim
in the HTML output, make sure that there are no blank lines in the
content of any of the HTML elements.

For example:

    <table>
      <tr>
        <th>Single *asterisk* or _underscore_</th>
        <th>Double **asterisks** or __underscores__</th>
      </tr>
      <tr>
    <td>Apply some _stress_</td>
    <td>Make it sound **important**</td>
      </tr>
    </table>

The text within the `td` tags in the above example shall not be
processed as vfmd, and shall be reproduced as-is in the HTML output.

Note that `pre`, `script` or `style` elements can have blank lines
enclosed within, and those blank lines don't cause the
verbatim-HTML-mode to end.

### HTML blocks within blockquotes and lists

When including a HTML block within a blockquote, please make sure that
the HTML block is completely contained within the blockquote.

For example, the following is wrong, and will result in invalid HTML
output:

    > This is a blockquote.
    > Below is a HTML block inside the blockquote.
    >
    > <div>
    > Inside the blockquote and inside the div block

    Outside the blockquote
    </div>

To correctly include the HTML block within the blockquote, please use
the `>` prefix for all the lines in the HTML block (or atleast for any
line following a blank line in the HTML block).

    > This is a blockquote.
    > Below is a HTML block inside the blockquote.
    >
    > <div>
    > Inside the blockquote and inside the div block
    >
    > Still inside the blockquote
    > </div>

Similarly, when including a HTML block within a list, please make sure
that the HTML block is completely contained within the list.

For example, the following is wrong, and will result in invalid HTML
output:

     1. This is a list.
        Below is a HTML block inside the list.

        <div>
        Inside the list and inside the div block

    Outside the list
    </div>

Instead, please indent all the lines of the HTML block (or atleast the
lines following a blank line) to line up with the list.

     1. This is a list.
        Below is a HTML block inside the list.

        <div>
        Inside the list and inside the div block

        Still inside the list
        </div>

### Matching open and close HTML tags

When you use HTML tags within the vfmd document, please take care to
correctly match up the open tags with the close tags. Any mismatched or
misnested HTML tags will remain mismatched or misnested in the output
HTML as well. Some implementations might clean up the invalid HTML, but
then their output might not really match what you intended in your
input. The best way to be clear about your intent is to provide
correctly matching HTML tags in the vfmd source document.

In addition, span-level vfmd markup will not be recognized
across mismatching or misnested HTML tags. 

For example, the asterisks in the following vfmd input are not
interpreted to mean strong emphasis:

    Cannot span **across <b><i>misnested</b></i> HTML tags** _at all_

For them to be recognized as vfmd markup, the HTML tags should be fixed
to nest correctly:

    Can span **across <b><i>correctly nested</i></b> HTML tags** _anytime_

So, when mixing HTML with vfmd, please ensure that the open HTML tags
and the close HTML tags you insert match up correctly.

