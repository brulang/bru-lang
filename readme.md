# Bru Lang

Bru is a simple markup language with [JSON](https://json.org)-like semantics.

It's currently used in [Bruno](https://www.usebruno.com) to save details of an api request in a file.

Here is a sample `.bru` file:
```groovy
http: {
  method: 'GET'
  url: 'https://www.usebruno.com/hello'
  headers: {
    Content-Type: 'application/json'
  }
  body: {
    type: 'xml'
    data: '''
      <xml>
        <name>Bru</name>
      </xml>
    '''
  }
}
```

## Design Goals
* Human readable
* Easy to represent multi line strings
* Support duplicate keys in dictionary - Multimap
* Indentation based syntax
* Annotations for providing additional information

The top level of a `.bru` file is an implicit Multimap. It does not require
or support braces.

Except where noted, blank lines and whitespace are ignored.

## Data Types

### Primitive Types
There are 4 primitive types in Bru.
* String
* Number
* Boolean
* Null

The string type slightly differs from JSON in that a Bru string is
a single-quoted and is always UTF-8. It may contain any printable UTF-8
character except `'` or `\n`. Whitespace is considered significant inside of the
quoted string.

### Composite Types
There are 3 composite types in Bru.
* [Multimap](https://en.wikipedia.org/wiki/Multimap)
* [Array](https://en.wikipedia.org/wiki/Array_data_structure)
* [Multiline String](https://en.wikipedia.org/wiki/Here_document)

#### Multimap
Multimap is a dictionary (key-value pair) with duplicate keys, enclosed in curly braces (`{}`). Keys and Values are separated by a colon `:` and key-value pairs are separated by a newline (`\n`).

All keys are unquoted strings and must not contain spaces. Values may be primitive or composite types. Some contexts may offer further restrictions not required by the Bru language.

```groovy
{
  name: 'Bru'
  social: {
    github: 'https://github.com/usebruno/bru-lang'
    twitter: 'https://twitter.com/use_bruno'
  }
}
```

#### Array
Array is a list of values separated by a newline (`\n`), enclosed in square brackets (`[]`).
```groovy
tags: [
  'markup langauge'
  'dsl'
]
```

#### Multiline String
Multiline string is a string that spans multiple lines, enclosed in triple single/double quotes (`'''`). The
content *must* begin on the line after the opening triple quote and the
closing triple quote *must* be on the line after the end of the multiline
string. Multiline strings must be indented two spaces more than the key
which contains them. This indentation is removed on parsing.

```groovy
article: {
  title: 'Bru Lang'
  content: '''
    Bru is a simple markup language with json like semantics.
    It's currently used in Bruno to save details of an api request in a file.
  '''
}
```

In the above example, the content will be:

```text
Bru is a simple markup language with json like semantics.
It's currently used in Bruno to save details of an api request in a file.
```

### Annotations
Annotations are used to provide additional information about a key-value pair. An annotation starts with (`@`) and ends with a newline (`\n`). Arguments may be passed to an annotation in parentheses (`()`) and separated by commas. Argument values may *only* be primitive types.

```groovy
http: {
  method: 'GET'
  url: 'https://www.usebruno.com/hello'
  headers: {
    Content-Type: 'application/json'

    @disabled
    @description('This is a sample request')
    Authorization: 'Bearer{{token}}'
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


### Comments

Comments start with `#` and end with a newline `\n`.
```bash
# This is a comment

http: {
  # This is a comment, too
}
```

### Original Authors
* [Anoop M D](https://github.com/helloanoop)
* [Ajai Shankar](https://github.com/ajaishankar)
- [Austin Ziegler](https://github.com/halostatue)