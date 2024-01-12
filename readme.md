# Bru Lang

Bru is a simple markup language with json like semantics.

It's currently used in [Bruno](https://www.usebruno.com) to save details of an api request in a file.

Here is how a sample `.bru` file looks like.
```groovy
http: {
  method: 'GET'
  url: 'https://www.usebruno.com/hello'
  headers: {
    Content-Type: 'application/json'
  }
  body: {
    type: 'xml'
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
* Annotations for providing additional information

## Data Types

### Primitive Types
There are 4 primitive types in Bru.
* String
* Number
* Boolean
* Null

A string can contain any character except `:` and `\n`

### Composite Types
There are 3 composite types in Bru.
* [Multimap](https://en.wikipedia.org/wiki/Multimap)
* [Array](https://en.wikipedia.org/wiki/Array_data_structure)
* [Multiline String](https://en.wikipedia.org/wiki/Here_document)

#### Multimap
Multimap is essentially a dictionary (key-value pair) that can have duplicate keys.
Its enclosed in curly braces `{}`.
Keys and Values are separated by a colon `:` and key-value pairs are separated by a newline `\n`.

All keys are strings. Values can be primitive types or composite types.

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
Array is a list of values separated by a newline `\n`. Its enclosed in square brackets `[]`.
```groovy
tags: [
  'markup langauge'
  'dsl'
]
```

#### Multiline String
Multiline string is a string that spans multiple lines. Its enclosed in parenthesis `()`.
```groovy
article: {
  title: 'Bru Lang'
  content: (
    Bru is a simple markup language with json like semantics.
    It's currently used in Bruno to save details of an api request in a file.
  )
}
```

### Annotations
Annotations are used to provide additional information about a key-value pair.
An annotation starts with `@` and ends with a newline `\n`.
Optionally, Comma seperated args can be passed to an annotation.

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
```

### Original Authors
* [Anoop M D](https://github.com/helloanoop)
* [Ajai Shankar](https://github.com/ajaishankar)