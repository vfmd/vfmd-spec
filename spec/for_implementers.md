# vfmd for implementers

This document describes the [vfmd Markdown syntax] in formal terms, and
is intended to be read by someone implementing this spec to parse or
otherwise programmatically interpret vfmd input.

[vfmd Markdown syntax]: introduction.md

This document is organized as follows: We start with a few definitions
that shall be used in the rest of this spec. Then, we discuss how
block-level vfmd elements may be identified, and then, we discuss how
span-level vfmd elements within a block-level element may be identified.

