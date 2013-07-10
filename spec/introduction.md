# Introduction

**vfmd** stands for Vanilla-flavoured Markdown. It formalises the
[original Markdown syntax] as a fully-defined specification. It also adds
a few (arguably minor) modifications to the syntax.

## Purpose

The core idea of the original Markdown has been adopted by a number of
implementations, but there is no complete formal specification of
Markdown yet (as of writing the vfmd spec) that clearly defines the
output for all possible inputs. This is also the reason why the
different Markdown implementations, while agreeing on the base syntax,
do not always agree on how to handle the interplay of different syntax
elements. This divergence implies that, unless written very carefully to
avoid any corner cases, a Markdown document is tied to the
implementation of Markdown that was used while writing the document.
Markdown would be a lot more valuable when different implementations in
different platforms agree on the output for all input scenarios, atleast
when using just the base syntax.

vfmd attempts to provide a specification that clearly defines the
Markdown output for all possible input scenarios, thereby enabling
different Markdown implementations that adopt it to behave consistently.

## Guiding Principles

The following are the principles that guide the design of the vfmd
specification, given in the order of their preference:

1. Stick to the goal of the original Markdown: "Make it as readable as
   possible". Keep in mind that "the biggest source of inspiration for
   Markdown’s syntax is the format of plain text email".
2. Make the output rich text structurally resemble the input plain text
3. Any input should be acceptable as valid Markdown
4. Stick to the [original Markdown syntax] as much as possible

[original Markdown syntax]: http://daringfireball.net/projects/markdown/syntax

