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

