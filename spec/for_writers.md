# vfmd for writers

This document describes the [vfmd] syntax in simple terms, and
is intended to be of help for people writing in vfmd.

The vfmd syntax is pretty much the same as the [original Markdown
syntax], with a few minor changes and added clarifications.

vfmd enables you to represent rich-text markup in plain-text files. It
uses punctuation characters as indicators of markup. It supports
block-level markup like lists, blockquotes and code-blocks, and also
supports span-level markup like emphasis, links and inline-code.

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

To create a setext-style header, follow the header text with an
line that "underlines" the text with equal signs (for first-level
headers) and dashes (for second-level headers).

For example:

    This is a first-level header
    ============================

    This is a second-level header
    -----------------------------

The "underline" line should start with a `=` or `-` character, can be of
any length, and can have interspersed space characters.

For example:

    This is a first-level header
    ==== == = =========== ======

    This is a second-level header
    ----

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

