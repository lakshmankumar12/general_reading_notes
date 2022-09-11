# links

https://www.tutorialspoint.com/yaml/yaml_collections_and_structures.htm

# Yaml features

AML includes a markup language with important construct, to distinguish
data-oriented language with the document markup. The design goals and features
of YAML are given below:

* Matches native data structures of agile methodology and its
  languages such as Perl, Python, PHP, Ruby and JavaScript
* YAML data is portable between programming languages
* Includes data consistent data model
* Easily readable by humans
* Supports one-direction processing
* Ease of implementation and usage

# Points to note

* yaml is case sensitive
* has `.yaml` as extension
* no tabs .. only spaces
* Indentation of white-space denotes block structure
* you can fold text to next line at the same intendation
* Multiple documents with single streams are separated with 3 hyphens (---).

## yet to understand

* Repeated nodes in each file are initially denoted by an ampersand `&`
  and by an asterisk `*` mark later.
* YAML always requires colons and commas used as list separators followed
  by space with scalar values.
* Nodes should be labelled with an exclamation mark `!` or double
  exclamation mark `!!`, followed by string which can be expanded
  into an URI or URL.

# Constructs

## lists

* list members have denoting hyphen `-`
    * hyphen+space to begin a new item in a specified list
* enclosed in square brace, sepearated by `,`
    * comma and space and the items are enclosed in JSON

```
--- # Favorite movies
 - Casablanca
 - North by Northwest
 - The Man Who Wasn't There

--- # Shopping list
   [milk, groceries, eggs, juice, fruits]

```

## Associative arrays

* Associative arrays are represented using colon ( : ) in the format of key value pair.
* They are enclosed in curly braces {}.

```yaml
- {name: John Smith, age: 33}
- name: Mary Smith
  age: 27
```

## Comments

* Begin with `#` character
* seperated from previous tokens by whitespace
