# Bru Lang (Specification Version 1.0)

Bru is a data definition language optimized for use with [Bruno][] to describe
an API request in a file.

Here is an example `.bru` file:

```hjson
http: {
  method: GET
  url: https://www.usebruno.com/hello

  headers: {
    Content-Type: application/json
  }

  body: {
    type: xml
    data: '''
      <xml>
        <name>Bru</name>
      </xml>
    '''
  }
}
```

Bru is:

- Human-readable. There is no "compact" representation. However there is
  a canonical format.

- Information-dense. The supported map structure is a [multimap][] allowing
  duplicate keys.

- Extensible. It supports an annotation format that consumers of Bru can use to
  enforce particular values or provide extra functionality.

The syntax has some similarities to [Hjson][].

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119][rfc2119].

## File Extension and Proposed Media Type

Bru language files have the media type `application/bruno+bru` (currently
unregistered).

Bru language files SHOULD have the extension `.bru`.

## Data Types

Bru supports four primitive data types and three composite data types.

- null
- boolean
- number
- string
- [multistring][]
- [array][]
- [multimap][]

```ebnf
VALUE ::= NULL | BOOLEAN | NUMBER | STRING | MULTISTRING | ARRAY | MULTIMAP
```

### Null

Null is an explicit non-value, written as `null`.

```ebnf
NULL ::= 'null'
```

### Boolean

There are two boolean values, `true` and `false`.

```ebnf
BOOLEAN ::= 'true' | 'false'
```

### Number

> [!TIP]
> In most contexts, bare numeric values will be treated as numbers. If the value
> `200` should be stored as a string, it MUST be quoted (`"200"` or `'200'`).

Numbers are integer or floating point values. Numbers MAY begin with a sign and
MUST contain at least one ASCII digit. Numbers MAY have a fractional part that
MUST start with the ASCII decimal point (`.`) and be followed by at least one
ASCII digit. Numbers MAY have an exponent part that MUST start with the ASCII
letter `e` or `E`, MAY have a sign, and MUST have at least one ASCII digit.
Numbers must match the regular expression
`^[-+]?[0-9]+(?:\.[0-9]+)?(?:e[-+]?[0-9]+)`.

```ebnf
DIGIT ::= '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'
SIGN ::= '-' | '+'
EXP ::= ('e' | 'E') SIGN? DIGIT+
INTEGER ::= SIGN? DIGIT+
FRACTION ::= '.' DIGIT+
NUMBER ::= INTEGER FRACTION? EXP?
```

All whitespace surrounding a numeric value SHALL be ignored. Whitespace within
a numeric value MUST prevent parsing the value as a number.

Bru parsers SHOULD retain the string representation of a number value and SHOULD
prefer the string representation of the value if there is a loss of fidelity. If
the parsed value cannot be reserialized without a loss of fidelity, it should be
retained as a string and the parsed value discarded.

> [!NOTE]
> In JavaScript, this usually means that the value is between
> `Number.MIN_SAFE_INTEGER` and `Number.MAX_SAFE_INTEGER`

### String

A Bru string may be quoted with paired single quotes (`'`) or double quotes
(`"`), but most strings do _not_ need to be quoted if they can unambiguously be
interpreted as a string.

If a string is unquoted, whitespace _around_ the string is trimmed, but
whitespace _within_ the string are not. That is,

```
   hello,  world
```

will become `hello,  world`.

Strings MUST be quoted if leading or trailing whitespace is significant, begin
with any of the characters `{}[],:'"#` or contains a comma (`,`).

> [!NOTE]
> If the string should be a string version of a number, `true`, `false`, or
> `null`, it MUST be quoted or the parser MAY treat the value as the primitive
> data type. That is, `key: null` means a key with a `null` value, but `key:
"null"` means a key with a string value of `"null"`.

Within quoted strings, the backslash (`\`) SHALL act as an escape character as
with JSON strings. Unquoted strings MUST NOT treat the backslash (`\`) as an
escape, so `\t` is treated as `'\\t'`.

### Multistring

Bru supports multi-line strings with either paired triple single quotes (`'''`)
or triple double quotes (`"""`). The contents of multistrings MUST be indented
an additional level, and the closing triple quotes MUST match the indentation of
the line containing the opening quotes.

Multistring quotes MUST NOT be followed by any non-whitespace character. Closing
multistring quotes MUST be preceded only by whitespace.

A Bru parser MUST suspend normal parsing until the matching closing triple quote
at the same indentation level is found. No Bru language constructs will be
matched within a multistring.

Within a multistring, the leading whitespace up to the correct indentation level
will be trimmed, but all other whitespace SHALL be preserved.

Examples for this should be clearer:

```
{
  key: '''
    the string begins two spaces more than key
    newlines are preserved, except for the
    last line. If the final newline is required,
    leave a blank line.

  '''

  array: [
    '''
      Multistrings are permitted in arrays,
      too.
        This indentation is kept, but not the newline.
    '''

    '''
      '''
        This contains a multistring within a multistring,
        but it is just a string.
      '''
    '''
  ]
}
```

The equivalent values in JSON would be:

```json
{
  "key": "the string begins two spaces more than key\nnewlines are preserved, except for the\nlast line. If the final newline is required,\nleave a blank line.\n",
  "extra": "extra line",
  "array": [
    "Multistrings are permitted in arrays,\ntoo.\n  This indentation is kept, but not the newline.",
    "'''\n  This contains a multistring within a multistring,\n  but it is just a string.\n'''"
  ]
}
```

#### Array

An array is a list of any Bru data type. It MUST be enclosed in square brackets
(`[`, `]`). Each entry MUST start on its own line, separated by newlines. String
values in an array follow [string](#String) quoting rules.

A Bru parser MUST ignore blank lines between entries (`null` values MUST be
explicitly entered).

Commas MAY be used to separate array entries written over multiple lines, but
their use MUST shift the parser so that commas MUST be used for all entries.
This permits table-style arrays:

An empty array MAY be written as `[]`.

```
{
  array: [
    1
    2
    3
    mixed values
    "[are okay]"

    {
      as: {
        are: [
          composite
          objects
        ]
      }
    }

    [
      or
      arrays
    ]

    null
  ]

  empty: []
}
```

#### Multimap

A dictionary allowing duplicate keys. It MUST be enclosed in curly braces
(`{}`). Key and value pairs are each placed on a new line with keys separated
from values by a single colon (`:`).

Keys MAY be unquoted if they match the pattern `^[_a-zA-Z][-_a-zA-Z0-9]*$`. All
other keys must be quoted. That is, `Content-Type` is a valid unquoted key, but
`"Content:Type"` or `"-Content-Type"` must be quoted.

Values MAY be any valid Bru data type. Composite data type delimiters _must_
follow the key.

Some application contexts (such as HTTP headers or query parameters) MAY apply
additional restrictions on valid keys or value types.

An empty multimap MAY be written as `{}`.

```hjson
{
  name: Bru
  social: {
    github: https://github.com/usebruno/bru-lang
    twitter: https://twitter.com/use_bruno
  }
}
```

The top level of a Bru document is an implicit multimap that does not require
braces.

```hjson
http: {
  headers: {
    # ...
  }
  query: {
    # ...
  }
}
```

The absence of a value following a key before the end of the line will result in
an empty string, making this:

```hjson
{
  name:
  social: {
  }
}
```

equivalent to:

```json
{
  "name": "",
  "social": {}
}
```

### Annotations

Annotations are used to provide additional information about a key-value pair
instance within a multimap.

An annotation MUST be on a single line preceding the key and MUST start with
`@` and end at the next newline. The name of an annotation MUST match the
pattern `^[_a-zA-Z][-_a-zA-Z0-9]*$`.

Annotations MAY have arguments which MUST be wrapped in parentheses (`()`).
Multiple arguments MUST be separated by commas `,`. Annotation arguments MUST be
primitive types (`null`, `boolean`, `string`, or `number`). Annotation string
arguments SHOULD be quoted.

```groovy
http: {
  method: GET
  url: https://www.usebruno.com/hello
  headers: {
    Content-Type: application/json

    @disabled
    @description(This is a sample request)
    Authorization: Bearer {{token}}
  }
  param: {
    query: {
      @description('The status of the user')
      @enum('active', 'inactive')
      status: 'active'
    }
  }
}
```

## Basic Formatting (Whitespace and Indentation)

Bru is block-oriented and uses both delimiters and indentation to ensure
correctness. When a new block context is started, the contents of the block MUST
be indented to the next level. Each level of indentation in Bru is signified
with two ASCII space characters (`0x20`).

Interior whitespace is not considered significant in Bru, except as noted for
[string](#String) and [multistring](#Multistring) values.

Blank lines MUST be ignored except as noted for [multistring](#multistring)
values.

Bru parsers MUST treat Windows line endings (`CRLF`) as equivalent to Unix line
endings (`LF`).

## Comments

Comments in Bru are line-oriented and MUST begin with the hash character (`#`),
which MUST be preceded only by whitespace: a comment MUST be on its own line.

If a comment marker is found on a line after a quoted string (`'string'` or
`"string"`) or after syntactically significant value markers (`:{}[]`), a parser
MUST treat that as a syntax error.

```
http: {
  # this is a comment
  key: value # this part of the string value
  extra: # this is not a comment and causes a parsing error
}
```

Leading or trailing whitespace for comments SHALL be adjusted by a Bru formatter
and a Bru parser MAY warn about comment-like lines.

## Serialization

Bru files SHOULD be serialized using platform native line endings (`CRLF` on
Windows, `LF` otherwise).

## Original Authors

- [Anoop M D](https://github.com/helloanoop)
- [Ajai Shankar](https://github.com/ajaishankar)
- [Austin Ziegler](https://github.com/halostatue)

[bruno]: https://www.usebruno.com
[hjson]: https://hjson.github.io
[json]: https://json.org
[multimap]: https://en.wikipedia.org/wiki/Multimap
[array]: https://en.wikipedia.org/wiki/Array_data_structure
[multistring]: https://en.wikipedia.org/wiki/Here_document
[rfc2119]: https://www.rfc-editor.org/rfc/rfc2119
