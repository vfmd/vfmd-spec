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

