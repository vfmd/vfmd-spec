# Introducing vfmd

**vfmd** is a variant of [Markdown] with a formal specification of its
syntax.

vfmd stands for _vanilla-flavoured markdown_. It formalises the
[original Markdown syntax] as a fully-defined specification, adding a
few (arguably minor) modifications to the syntax in the process.

[Markdown]: http://daringfireball.net/projects/markdown/
[original Markdown syntax]: http://daringfireball.net/projects/markdown/syntax

## Background

The [original Markdown syntax] document is a set of loosely defined
guidelines on what the syntax elements in Markdown are, and how they
should be interpreted. <span id="core-syntax">The syntax elements
specified in the [original Markdown syntax] constitute the **core
syntax** of Markdown.</span>

Since the original Markdown was published in 2004, there have been many
different implementations of Markdown, written in different programming
languages, and many different flavours of Markdown, with additional or
modified features or syntax elements.  All these different
implementations and flavours of Markdown are based on the _core syntax_
as defined in the [original Markdown], but their interpretations have
[not always been consistent][babelmark2], even when we consider just the
_core syntax_. This divergence implies that, unless written very
carefully to avoid any corner cases, a Markdown document is tied to the
variant of Markdown that was used while writing the document.

[original Markdown]: http://daringfireball.net/projects/markdown/syntax
[babelmark2]: http://johnmacfarlane.net/babelmark2/faq.html

## Purpose

Markdown would be a lot more valuable when different implementations and
flavours agree on the output for all input scenarios, atleast when using
just the _core syntax_.

One of the main reasons why the different variants of Markdown differ in
their behaviour is that there is no real specification for Markdown.  In
the absense of a specification, the developers behind the different
variants of Markdown have interpreted the loosely defined guidelines
specified in [original Markdown syntax] in different ways, thereby
resulting in this divergence.

vfmd attempts to provide a well-defined specification for the _core
syntax_ of Markdown. vfmd shall clearly define the interpretation (and
hence, the output) for all possible input scenarios, thereby enabling
different Markdown implementations that adopt it to behave consistently
in interpreting the _core syntax_ of Markdown.

## Goals

These are the goals for vfmd:

 1. vfmd shall unambiguously define the interpretation for all input
    scenarios for the _core syntax_ of Markdown
 2. In case a vfmd implementation wants to support any additional syntax
    elements not covered by the _core syntax_ (e.g., fenced code-blocks,
    footnotes), vfmd shall define how the handling of the custom
    additional syntax should be integrated with the handling of the
    _core syntax_

## Guiding Principles

The following are the principles that guide the design of the vfmd
specification, given in the order of their preference:

1. Stick to the goal of the original Markdown: "Make it as readable as
   possible". Keep in mind that "the biggest source of inspiration for
   Markdown’s syntax is the format of plain text email".
2. Make the output rich text structurally resemble the input plain text
3. Any input should be acceptable as valid Markdown
4. Stick to the [original Markdown syntax] as much as possible

