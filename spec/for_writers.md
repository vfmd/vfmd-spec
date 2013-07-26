# vfmd for writers

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

The expressive power of vfmd is rather limited compared to plain old
HTML. However, if you are capable of authoring HTML documents, you can
use snippets of HTML in your vfmd document to augument the vfmd feature
set. That said, if you're planning to convert the vfmd document to a
format other than HTML, the support for using HTML tags in the vfmd
document might be limited or absent, depending on the implementation.

The following HTML elements are handled specially in vfmd: `address`,
`article`, `aside`, `blockquote`, `div`, `dl`, `fieldset`, `form`, `hr`,
`nav`, ` noscript`, `ol`, `pre`, `section`, `table`, `ul`. Let's call
them _HTML grouping elements_. (These are basically the [HTML4 block
elements][HTML4-block], except the paragraph element `p` and the heading
elements `h1` to `h6`, put together with the [HTML5 sectioning content
elements][HTML5-sectioning]).

[HTML4-block]: http://www.w3.org/TR/html4/sgml/dtd.html#block
[HTML5-sectioning]: http://www.w3.org/html/wg/drafts/html/master/dom.html#sectioning-content

### Verbatim HTML

If you want the vfmd input to include a HTML block that should appear
verbatim in the HTML output, start the HTML block with a _HTML grouping
element_ and do not include any blank lines in the content of any of the
HTML elements.

For example:

    <table>
      <tr>
        <td>One *asterisk*</td>
        <td>Two **asterisks**</td>
      </tr>
      <tr>
        <td>One _underscore_</td>
        <td>Two __underscores__</td>
      </tr>
    </table>

The text within the `td` tags in the above example shall not be
processed as vfmd, and shall be reproduced as-is in the HTML output.

### vfmd within HTML elements

If you want some text within the HTML elements to be treated and
processed as vfmd, separate out the vfmd-processable part from the
verbatim-HTML part with one or more blank lines. The vfmd-processable
parts should not start with a _HTML grouping element_. Also, please make
sure that the line following the separating blank line is not indented
by more than 3 spaces (if it's indented by 4 or more spaces, it would
become a code block).

For example, for the input:

    <table>
      <tr>
        <td>One *asterisk*</td>
        <td>Two **asterisks**</td>
      </tr>
      <tr>

    <td>One _underscore_</td>
    <td>Two __underscores__</td>

      </tr>
    </table>

The corresponding HTML output shall be:

    <table>
      <tr>
        <td>One *asterisk*</td>
        <td>Two **asterisks**</td>
      </tr>
      <tr>

    <td>One <em>underscore</em></td>
    <td>Two <strong>underscores</strong></td>

      </tr>
    </table>

You can also use HTML tags inlined within the text of a vfmd paragraph.
However, if you use any _HTML grouping element_ inside a vfmd paragraph,
vfmd syntax will not be recognized within the _HTML grouping element_.

For example:

    This paragraph has **bold**, *italics* and
    <s>strikethrough _text_</s>. When we add a <div> here, we can 
    no longer use vfmd for *emphasis* or [linking](http://example.net),
    until we close the </div>, _or_

    start a **new** paragraph.

results in the following HTML output:

    This paragraph has <strong>bold</strong>, <em>italics</em> and
    <s>strikethrough <em>text</em></s>. When we add a <div> here, we can 
    no longer use vfmd for *emphasis* or [linking](http://example.net).
    until we close the </div>, <em>or</em>

    <p>start a <strong>new</strong> paragraph.</p>

You might have noticed that in this example, the second paragraph of
text is wrapped in `p` tags, while the first paragraph is not. Also, in
the previous example, the `td` elements were not wrapped in `p` tags.

vfmd skips wrapping a paragraph of text in `p` tags if any of the
following is true:

 1. The paragraph contains one or more _HTML grouping elements_
 2. There are one or more unmatched tags (i.e. open tag without a
    corresponding close tag, or vice versa) in the paragraph
 3. There is a HTML tag (open tag or close tag or self-closing tag) at
    both the beginning and the end of the paragraph

In case any of the above is true, but you still want it become a HTML
paragraph, you will have to supply the open and close `p` tags yourself.

For example, consider the vfmd paragraph:

    <del>Deleted content at the _beginning_.</del> Intact content
    in the _middle_. <ins>Added content at the _end_.</ins>

Since there's a HTML tag at both the beginning (`<del>`) and end
(`</ins>`) of the paragraph, it will not be automatically wrapped in `p`
tags. For it to become a paragraph in HTML, it should we wrapped in `p`
tags in the vfmd input itself:


    <p><del>Deleted content at the _beginning_.</del> Intact content
    in the _middle_. <ins>Added content at the _end_.</ins></p>

### Special cases

As we said earlier, blank lines can be used to separate the
verbatim-HTML parts from the vfmd-processable parts. However, in the
following situations, a blank line does not break a verbatim HTML block:

 1. When the blank line is within a HTML tag, like:

        <p class="wrapper"

           style="color: #555;">
        </p

        >

 2. When the blank line is within a HTML comment, like:

        <!-- There is a blank line below

        There is a blank line above -->

 3. When the blank line is contained within a `pre`, `script` or `style`
    HTML element, like:

        <pre>

        The blank line above and the blank line below
        don't break this verbatim HTML block

        </pre>

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
correctly match up the open tags with the close tags. If the vfmd
document contains unmatched tags of _HTML grouping elements_, those
mismatches will be present in the output HTML as well, resulting in the
output being invalid HTML. If the vfmd document contains unmatched tags
of other HTML elements, the output HTML will not have the mismatch, but
how it turns out in the output HTML might not be what you intended.

So, when mixing HTML with vfmd, please ensure that the open HTML tags
and the close HTML tags you insert match up correctly.

