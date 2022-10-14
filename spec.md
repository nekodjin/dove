The markup language used by Dove is called DoveML. This document specifies
DoveML and the file format thereof.

DoveML files are required to have a name that ends in `.dml`. They are also
required to be valid utf8-encoded text. DoveML files are interpreted as
utf8-encoded text.

It is highly recommended that any one line should not exceed 80 characters in
length. However, this recommendation is not enforced - neither with errors nor
with warnings - as there is no satisfactory way to define the length of a line.

DoveML files consist of a combination of plain text and macro invocations.
Macro invocations are replaced by any combination of formatted text, images,
videos, files, links, etc, which is determined by the definition of the
particular macro being invoked.

Line separators are characters or sets of characters that are used to delimit
separate lines. Line separators are used in the definition of DoveML, so they
are first defined here:

A line separator is the character CR, optionally followed by the character LF,
or the character LF. CR refers to the Unicode character U+000D, and LF refers
to the character U+000A. A line separator is the longest possible sequence of
characters matching this definition that is immediately preceded by a character
that is not CR or LF, or by another line separator. DoveML files must not
contain any CR or LF characters that are not part of a line separator.

"Whitespace characters" are another class of characters that are referred to by
the DoveML definition. The term whitespace character(s) refers to any Unicode
characters with the White_Space property, with the exception of any Unicode
characters in the Control category, as well as TAB (U+0009).

It is illegal for any Unicode character in the Control category, other than TAB
(U+0009), to appear in a DoveML file.

Any text not constituting a macro invocation (or any body text of a macro
invocation, in lieu of conflicting macro effects) is subject to the following
modifications, in order. These are termed the "ordinary formatting rules".

1.  Any sequence of line separators, interrupted only by uninterrupted
    sequences of whitespace characters, will be replaced by a sequence of
    uninterrupted line separators whose length is one less than the length of
    the original sequence.

2.  Any uninterruped sequence of whitespace characters will be replaced by a
    single SPACE (U+0020) character.

The syntax of a macro invocation consists of a `{` symbol followed by exactly
one `%` character. If more than one `%` character follows the opening brace,
it does not begin a macro invocation but is rather replaced by the literal text
`{`, followed by one less than the number of `%` characters that followed the
opening `{`. For example, `{%%` is replaced by the literal text `{%`. Following
the opening brace is a raw invocation, most concisely described by the annotated
grammar below. The macro invocation is then closed by any number of whitespace
characters, followed by exactly one `%` and exactly one `}`. If there is more
than one `%` character immediately preceding the `}` character, similar rules
apply as with the opening brace: it does not close the macro invocation, but
rather is replaced by the literal text of one less than the number of `%`
characters that occurred before the `}` character, followed by a `}` character.
For example, `%%}` is replaced by the literal text `%}`, rather than closing
a macro invocation.

The grammar of a raw invocation is as follows:

```
RAW_INVOCATION
  = { WHITESPACE_CHARACTER }
    MACRO { "," { WHITESPACE_CHARACTER } MACRO } [ "," ]
    { WHITESPACE_CHARACTER }
    [ ":" { WHITESPACE_CHARACTER } MACRO_BODY ]
  ;

MACRO
  = MACRO_NAME
    [ "(" MACRO_ARGUMENT ")" ]
  ;
```

Square brackets (`[]`) indicate that the production rule within may occur
zero or one times. Curly braces (`{}`) indicate that the production rule
within may occur any number of times (including zero). Quotation marks (`""`)
indicate that the text within refers to those literal characters within.

`WHITESPACE_CHARACTER` indicates any character defined herein as a "whitespace
character". `MACRO_NAME` may be any of the macro names listed further down this
document. Syntactically, `MACRO_ARGUMENT` may be any text that would in
isolation constitute valid contents of a DoveML file, so long as it contains
an equal number of `(` characters as `)` characters. However, specific macros
may enforce additional restrictions on the value of this grammatical term. A
particular macro may also require that the optional clause in which the
`MACRO_ARGUMENT` appears must or must not be present. Similarly, `MACRO_BODY`
may be any text that would in isolation constitute valid contents of a DoveML
file; however, a particular macro may enforce additional requirements on the
value of this grammatical term, as well as requiring that the optional clause
in which it occurs must or must not exist. Neither `MACRO_ARGUMENT` nor
`MACRO_BODY` are permitted to be zero Unicode characters long, if their
respective clauses are present.

In lieu of any additional requirements imposed by a particular macro, the
macro body may contain additional macros. These macros are said to be "child"
macros of the enclosing macro, and the enclosing macro is said to be a "parent"
macro of those child macros. The phenomenon as a whole is termed "nested
macros".

Specific macros may also mandate specific properties about the context - that
is to say, the surrounding text or parent macro(s) - in which they are invoked.
It is illegal for these any of the mentioned requirements to be violated. It is
also illegal for a DoveML file to contain any incomplete macro invocations -
that is to say, any `{%` macro openings not matched by a `%}` macro closing, or
vice versa, or any matching pairs of macro openings and macro closings whose
contents do not match the grammar of a raw invocation.

So long as it does not violate the rules presented above, the text surrounding
a macro invocation may be any sequence of Unicode characters, of any length.

If the repetition clause containing `MACRO` is repeated more than zero
times, it will be removed and the macro body will be replaced by an invocation
of that clause, with the previous macro body as the body thereof. For example,
a macro invocation of the form `{% a, b, c : xyz %}` will be evaluated as if it
were `{% a : %{ b : {% c : xyz %} %} %}`.

The list of macros and their properties are as follows:

- Macro name: `bold`

  Requirements:

  - There must not be a `MACRO_ARGUMENT` clause.

  - There must be a `MACRO_BODY` clause.

  Effect:

  Formats the body text in bold face.

- Macro name: `ital`

  Requirements: see `bold`

  Effect:
  
  Formats the body text in italic font face. (overrides a parent `obliq` macro)

- Macro name: `obliq`

  Requirements: see `bold`
  
  Effect:
  
  Formats the body text in oblique font face (overrides a parent `ital` macro)

- Macro name: `mono`

  Requirements:
  
  - The argument clause must not exist, or else the argument must be the
    identifier of one of the recognized languages for syntax highlighting.
  
  - There must be a body clause.
  
  Effect:
  
  Formats the body text in a monospaced font. Child macro invocations are
  permitted only if they are invocations of `bold`, `ital`, `obliq`, or `link`.
  Child macros may not contain child macros. The ordinary formatting rules will
  not be applied to the body. If an argument is present, the body text will be
  colored in accordance with the syntax highlighting engine for the appropriate
  language.
  
- Macro name: `code`

  Requirements: see `mono`
  
  Effect:
  
  Formats the body text in a monospaced font in a visually distinct,
  line-breaking block of text. Child macro invocations are not permitted. The
  ordinary formatting rules will not be applied to the body. Any sequence of
  line separators interrupted only by whitespace characters that occurs either
  at the very beginning or the very end of the body will be removed entirely.
  If an argument is present, the body text will be colored in accordance with
  the syntax highlighting engine for the appropriate language.

- Macro name: `link`
  
  Requirements:
  
  - An argument clause must be present.
  
  - The argument must be an absolute path, a relative path, or a URL.
  
  - There must be a file at the location specified by the argument.
  
  Effect:
  
  If there is no body, the argument will be treated as the body. The body will
  be highlighted in a medium shade of blue and, when hovered over with a mouse
  or selected using tab navigation, be underlined. If it is clicked with a mouse
  or if the enter key is pressed after it is selected using tab navigation,
  the viewer of the document will be redirected to the link specified by the
  argument.

- Macro name: `image`

  Requirements: see `link`; the file specified by the argument must be an image
  file; if there is a body, it must not contain any child macros.
  
  Effect:
  
  The macro will be replaced by the image specified by the argument. If there is
  a body, it will be treated as the alt text for the image.

- Macro name: `video`

  Requirements: see `link`; the file specified by the argument must be a video
  file; if there is a body, it must not contain any child macros.
  
  Effect:
  
  The macro will be replaced by the video specified by the argument. If there is
  a body, it will be treated as the alt text for the video.

- Macro name: `list`

  Requirements:
  
  - There must not be an argument clause.
  
  - There must be a body.
  
  - The body may contain only child invocations of the `elem` macro, line
    separators, or whitespace characters.
  
  - No child macro may have an argument clause, or all child macros must contain
    an argument clause with a numeric argument, or all child macros must contain
    an argument clause with an alphabetic argument.
  
  Effect:
  
  Creates a list with the elements specified by the child macros. Line
  separators and whitespace characters in the body are removed.

- Macro name: `elem`
  
  Requirements: none
  
  Effect:
  
  If there is no argument, this creates a list element delimeted by a bullet
  point. If there is a numeric argument (an argument consisting only of a
  sequence of characters from 0 (U+0030) to 9 (U+0039) of any non-zero length),
  or if there is an alphabetical argument (an argument consisting only of a
  sequence of characters from A (U+0041) to Z (U+005A) of any non-zero length),
  then this creates a list element delimeted by the argument. If there is a
  body, then the body forms the body of the list element.

The list of supported languages and their identifiers is as follows:

There are currently no supported languages for syntax highlighting.
