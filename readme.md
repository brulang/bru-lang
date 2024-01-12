# Bru Lang

Bru is a simple markup language with json like semantics.

It's currently used in [Bruno](https://www.usebruno.com) to save details of an api request in a file.

Here is how a sample `.bru` file looks like.
```groovy
http: {
  method: GET
  url: https://www.usebruno.com/hello
  headers: {
    Content-Type: application/json
  }
  body: {
    type: xml
    data: (
      <xml>
        <name>Bru</name>
      </xml>
    )
  }
}
```

## Design Goals
* Human readable
* Easy to represent multi line strings
* Support duplicate keys in dictionary - Multimap
* Indentation based syntax


## Data Types

### Primitive Types
There is only one primitive type in Bru - String.
Its upto the consumer to interpret the string as a number, boolean, date, etc.

A string can contain any character except `:`, `\n`

### Complex Types
There are 3 complex types in Bru.
* [Multimap](https://en.wikipedia.org/wiki/Multimap)
* [Array](https://en.wikipedia.org/wiki/Array_data_structure)
* Multiline String

#### Multimap
Multimap is essentially a dictionary (key-value pair) that can have duplicate keys.
Its enclosed in curly braces `{}`.
Keys and values are separated by a colon `:` and key-value pairs are separated by a newline `\n`.

All keys are strings. Values can be multimap, array or multiline string.
```groovy
{
  name: Bru
  social: {
    github: https://github.com/usebruno/bru-lang
    twitter: https://twitter.com/use_bruno
  }
}
```

#### Array
Array is a list of values separated by a newline `\n`. Its enclosed in square brackets `[]`.
```groovy
[
  1
  2
  3
]
```

#### Multiline String
Multiline string is a string that spans multiple lines. Its enclosed in parenthesis `()`.
```groovy
(
  This is a multiline string.
  It can span multiple lines.
)
```

### Comments
Comments start with `#` and end with a newline `\n`.
```groovy
# This is a comment
```

### Original Authors
* [Anoop M D](https://github.com/helloanoop)
* [Ajai Shankar](https://github.com/ajaishankar)