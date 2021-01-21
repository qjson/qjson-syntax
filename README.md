# Syntax of QJSON files

version: v0.0.0

QJSON files are intended to be created and maintained by humans to 
provide input data to programs. The expected most frequent usage is
configuration files. Functions are provided to convert a QJSON text
into JSON for which many libraries exist.

This project specifies the QJSON syntax. Since QJSON is young, it may
still evolve. It is the right time to make suggestions or corrections
if you have any. 

## Syntax

### Whitespace

The white space characters are: 
1. ' ' (classical white space: " ")
2. '\t' (tab: "\t")
3. nbsp (non breaking space: "\u00A0")

### Character

A character is any valid Unicode character, but with control characters excluded.

### Comments

QJSON files may contain comments. There are three types of comments that
should be familiar to programmers. When converting QJSON to JSON, comments 
are discarded.

    #...<nl> 
    //...<nl>
    /*...*/

Comments may contain any characters. The /*...*/ comment may also contain 
control characters.

### Strings

QJSON has four types of strings:
1. double quoted string (same as JSON strings),
2. single quoted string (same as double quoted, but single quoted),
3. quoteless string
4. multiline string

Identifiers can be double or single quoted strings or quoteless strings. 
Multiline strings can only be values. 

#### Single and double quoted strings

Single and double quoted string may contain any character and escaped
characters. The / character doesn’t need to be escaped. The QJSON to JSON
converter will escape the / character for you when it is needed. A single 
quoted string doesn’t need to escape ", but must escape ' when needed.

#### Quoteless strings

A quoteless string will start at a non whitespace character and will end 
at a delimiter (:,{}[]), the start of a comment or a newline. The column
in an ISO 8601 date time value will not terminate the quoteless string. 
The ' and " characters in a quoteless string don’t need to be escaped. 
A quoteless string may not contain escaped characters. When converted 
to JSON string, the `\` character is escaped (e.g. `\b` -> `\\b`). A 
quoteless string is right trimmed of whitespaces. 

Examples:

    The brown fox , --> "The brown fox"
    my id :   --> "my id"
    x \n y #...  --> "x \\n y"

#### Multiline strings

A multiline string must start on a newline and is delimited with the character 
\` at both ends. Multiline strings can have a margin to support clean alignment.
The margin is defined by the sequence of whitespace characters in front of the 
starting \` character. All subsequent lines of the multiline string must have 
the exact same margin. 

The starting \` *must* be followed by a newline type specifier (\n or \r\n). It  
specifies the newline value to use in the JSON string. It is needed because
editors may change the type of newline (\n or \r\n) in the QJSON file according
to the operating system convention or configuration settings. Specifying the 
newline type ensures 100% determinism of the JSON output. The newline type 
specification may be followed by spaces or a comment. 

The multiline string content starts on the line after the starting \` and newline
specification.

A multiline string may contain unescaped control characters. They will be escaped
in the JSON string. A multiline string may contain a \` character, but it must 
be escaped by adding a \ character *just after it*. The \ character will not be 
present in the JSON string. 


Examples:

    `\n
    The
    brown
    fox`

    --> "The\nbrown\nfox"

       `  \r\n # this multiline has two white space margin
       The
       brown
       fox`

    --> "The\r\nbrown\r\nfox"

    ` \n // multiline containing a ` character
    Text `\example\`
    `
  
    --> "Text `example`\n"

### Literal constants

Literals constants are the well known `true`, `false` and `null` values of JSON. 
QJSON has aliases for these values that will be replaced by the corresponding JSON
value.

The aliases of `true` are `True`, `TRUE`, `on`, `On`, `ON`, `yes`, `Yes`, `YES`.

The aliases of `false` are `False`, `FALSE`, `off`, `Off`, `OFF`, `no`, `No`, `NO`. 

The aliases of `null` are `Null` and `NULL`. 

### Numbers

Numbers can be the classical integer and decimal number of JSON with the addition 
that they may contain _ characters to separate groups of digits. A number can’t start
or end with an _ and ther can’t be more than one _ in sequence. 

QJSON also supports binary numbers starting with `0b` or `0B` and followed by 0 and 1.
Binary numbers may also contain _ characters.

Hexadecimal numbers starting with `0x` or `0X` and octal numbers starting with `0o` or
`0O` are also supported. Octal numbers without the `o` or `O` are also allowed (e.g. 0750).

QJSON supports also simple mathematical expression. They will be evaluated in the 
conversion and the result will be output to JSON. The expression supports the operation
+-/* on integers and decimals, and |&^~ on integers only. Parenthesis can be used for grouping.

QJSON uses implicit casting from integer to float when needed. Casting from int to float is 
not allowed. 

The translator parses numbers and expressions as quoteless strings. It is in a second 
step that the quoteless string is interpreted as a numeric expression when possible. The
consequence is that two numbers in sequence (e.g. "15 30") will be read as one quoteless
string. Since it starts by a digit it will be considered to be a number and interpreted as
such. But two numbers in sequence is not a valid expression. This will thus generate an error.
To avoid this, add a comma between the numbers to tell the translator that these are two 
distinct numbers (e.g. "15, 30"). 

### Objects
7that an identifier may contain spaces. Multiline 
strings can’t be used as identifiers.

When a value is an object, it is enclosed in the {} characters as in JSON. 

### Arrays

Array values are enclosed in [] as in JSON. The , separating values is optional. 

Beware of the quoteless strings with multiple values on the same line. Quoted 
strings will be recognized as a single value and don’t need to be separated by
commas. Numbers and literal constants must be separated by commas, otherwise 
they will be treated as a big quoteless string. 


## Examples

### Example 1

QJSON :

    # my QJSON file
    a : 123
    b c : 0b_1101_1111 | 0x20
    d : The brown fox...

JSON : 

    {"a":123,"b c":255,"d":"The brown fox..."}

### Example 2

QJSON :

    // my QJSON file
    text :
        ` \n // multiline
        The
        brown
        fox
        `
    v : [1, 2, 3, 4
         5
         hi there /* why not */
         7*2
         ]
    10 : 3.1415 * 2

JSON : 

    {"text":"The\nbrown\nfox\n","v":[1,2,3,4,5,"hi there",14],"10":6.283}
