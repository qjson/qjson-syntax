# Syntax of QJSON files

version: v0.0.0

QJSON files are intended to be created and maintained by humans to 
provide input data to programs. The expected most frequent usage is
configuration files. Functions are provided to convert a QJSON text
into JSON for which many libraries exist.

This project specifies the QJSON syntax. Since QJSON is young, it may
still evolve. It is the right time to make suggestions or corrections
if you have any. 

Examples of QJSON text and the resulting JSON text:

### Example 1
#### QJSON:

```
planet : Earth
diameter : 12 742 km
diameter value : 12742
diameter unit : km
```
#### JSON:

{"planet":"Earth","diameter":"12 742 km","diameter value":12742,"diameter unit":"km"}

### Example 2

#### QJSON:

```
value: 123_456 * 2         # underscore in numbers and evaluated arithmetic expressions
some bits: 0b0110_1110     # binary numbers with underscore
a real value: -.1_2_3E1_3  # real value with underscores
octal value : 0732 + 0o732 # octal values
hexadecimal : 0x10 / 2     # hexadecimal values
```
#### JSON:

```
{"value":246912,"some bits":110,"a real value":-1230000000000,"octal value":948,"hexadecimal":8}
```

### Example 3

#### QJSON

```
array: [1, 2, 3, hello world ]
object: {a: 1, b:2, c:this and this}
```

#### JSON

```
{"array":[1,2,3,"hello world"],"object":{"a":1,"b":2,"c":"this and this"}}
```

### Example 4

#### QJSON

```
duration: 2h 30m                  # will be converted to seconds
stamp: 2019-02-13T01:10:32.123456 # will be converted to UTC seconds since 1970-01-01:00:00:00.000000
```

#### JSON

```
{"duration":9000,"stamp":1550020232.123456}
```

### Example 5

#### QJSON

```
the text : 
           ` \r\n # multiline string with a skipped left margin
           Hello world !
           
           This is a multiline
           example containing a `\.
           `
```

#### JSON

```
{"the text":"Hello world !\r\n\r\nThis is a multiline\r\nexample containing a `.\r\n"}
```

## Syntax

### Whitespace

The white space characters are: 
1. ' ' (classical white space: " ")
2. '\t' (tab: "\t")
3. nbsp (non breaking space: "\u00A0")

### Character

A qjson character is any valid Unicode character with control characters 
except the tab character excluded. The tab character is thus in the set
of qjson characters. 

In the following documentation, when we write "any characters", it means
any unicode character with control characters except tabs excluded. When 
control characters are also allowed, it is explicitely specified.

### Comments

QJSON files may contain comments. There are three types of comments that
should be familiar to programmers. The conversion of QJSON to JSON ignores
comments.

    #...<nl> 
    //...<nl>
    /*...*/

All the characters after `#` or `//` in a line are treated as comments. 
These comments may contain any characters (control characters are not
allowed).

A `/*...*/` comment starts at `/*` and ends at `*/`. It may contain 
any character and control characters. It may thus span multiple lines. 

### Strings

QJSON has four types of strings:
1. double quoted string (same as JSON strings),
2. single quoted string (same as double quoted, but single quoted),
3. quoteless string
4. multiline string

All string types can be used as values. All string types except the 
multiline string can be used as identifier. 

#### Double quoted string

Double quoted strings are exactly like JSON strings. It contains any
characters and escaped characters. It is delimited by `"` and 
contained `"` must be escaped as `\"`. The `/` doesn’t need to be escaped.
It will be automatically escaped by the conversion when needed. 

#### Single quoted string

Single quoted string is like a double quoted string with the difference
that it is delimited by `'`. Internal single quotes must be escaped and
double quotes must not be escaped. As for the double quoted string, the
characters `/` will be automatically escaped when needed. 

#### Quotelesss string

Quoteless strings don’t need the `"` or `'` delimiter and don’t need to
escape them. From this perspective, they are more convenient to use and
more readable. A quotless string starts at the first non space character
and ends a the last non space character before the characters `:`, `,`, 
`{`, `}`, `[`, `]`, a start of comment or a newline. 

A quoteless string can’t contain control characters or escaped 
characters. A `\b` will be treated as the character `\` followed by the 
character `b`. 

#### Multiline string

A multiline string must start on a newline and is delimited by the character `\`` 
at both ends. The starting `\`` must be followed by a newline type specifier that
is `\n` or `\r\n`. Newlines in the multiline string will be replaced by this 
newline type. This is required because depending on the OS or configuration editors 
may change the newline in the qjson text. By specifying the newline to use in the
JSON string, we ensure a fully deterministic behavior of the conversion. The 
newline type specifier may be followed by a `#` or `//` comment, not by a `/*...*/` 
comment. 

The multiline string content starts on the next line and ends at the trailing `\``. 

Multiline strings may have a margin that will be stripped off in the conversion 
process. The margin is defined by the sequence of whitespace characters
in front of the starting `\`` character. All subsequent lines of the multiline
string must have the exact same margin. The margin can be made of any combination
of whitespace characters (` `, non breaking space, `\t`). 

A multiline string may contain a `\`` character, but it must be escaped by adding 
a `\\` just after it. The `\\` character will be removed in the conversion process. 


Multiline strings may contain control characters. They will be escaped in the
JSON string. 

### Literal constants

Literals constants are the well known `true`, `false` and `null` values of JSON. 
QJSON has also aliases for these values that will be replaced by the corresponding 
JSON value.

The aliases of `true` are `True`, `TRUE`, `on`, `On`, `ON`, `yes`, `Yes`, `YES`.

The aliases of `false` are `False`, `FALSE`, `off`, `Off`, `OFF`, `no`, `No`, `NO`. 

The aliases of `null` are `Null` and `NULL`. 

### Numbers

Numbers can be the classical integer and decimal number of JSON with the addition 
that they may contain `_` characters to separate groups of digits. A number can’t start
or end with an `_` and ther can’t be more than one `_` in sequence. 

QJSON also supports binary numbers starting with `0b` or `0B` and followed by one or more
0 or 1. Binary numbers may also contain `_` characters.

Hexadecimal numbers starting with `0x` or `0X` and octal numbers starting with `0o` or
`0O` are also supported. Octal numbers without the `o` or `O` are also allowed (e.g. 0750).

QJSON supports also simple arithmetic expressions that will be evaluated in the conversion 
and the result will be output to JSON. The expression supports the operations `+`, `-`, `/` 
and `*` on integers and decimals, and `|`, `&`, `^` and `~` on integers only.

QJSON uses implicit casting from integer to float when needed. Explicit casting from integer
to float is not supported. 

The conversion first parses numbers and expressions as quoteless strings. If the resulting
string starts with a digit, skipping spaces, `+`, `-` and any number of `(`, the string is 
assumed to be a numeric expression and interpreted as such. The consequence is that two 
numbers in sequence (e.g. "15 30") will be parsed as one quoteless string, then assumed 
to be a numeric expression, and finaly detected as an invalid numeric expression. It will 
thus generate an error. You may want to add quotes if it is to be interpreted as a string, 
or a comma between the numbers if it is to be interpreted as a sequence of two numbers. 

### Objects

As in JSON, QJSON objects are delimited by `{` and `}`. Objects may be encapsulated. 
Objects contain any number of identifier and value pairs separated by `:`. An object
can only appear as value as in JSON.  

### Arrays

As in JSON, QJSON array values are enclosed in `[` and `]`. The `,` separating values 
is optional. It is required to separate numbers, numeric expressions, or quoteless 
strings when on the same line.
