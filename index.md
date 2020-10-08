# 1. About
[RFC8259][1] describes the JSON interchange format, which is widely used in
application-level protocols including RESTful APIs. It is common for
applications to request resources via the HTTP POST method with JSON entities
because, per RFC7231
[Section 4.3.1](https://tools.ietf.org/html/rfc7231#section-4.3.1):

    A payload within a GET request message has no defined semantics;
    sending a payload body on a GET request might cause some existing
    implementations to reject the request.
 
However, POST is suboptimal for requests which do not modify a resource's
state because it is not idempotent and limits or prevents client caching.

While one could simply [percent encode][8] JSON text such that it's suitable
for inclusion in an HTTP GET request, anything other than a trivial payload
would be difficult for a human to read or modify; that's presumably why it's
not often done in practice.  Alternatively, one could use something other than
JSON, however, there's good reason to choose JSON. It defines a simple but
powerful data model, and it's very well known.

JSON&#x2192;URL defines a text format for the JSON data model suitable for
use within a [URL][10]/[URI][4].

## 1.1 Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
[RFC2119][3] when, and only when, they appear in all capitals, as shown here.
   
The terms "JSON text", "value", "object", "array", "number", "string",
"name", and "member" in this document are to be interpreted as described
in [RFC8259][1].

## 1.2 Requirements Notation
This document uses the Augmented Backus-Naur Form (ABNF) notation described
in [RFC5234][2].

## 1.3 Latest Version
The latest version of this document may be found here:
https://github.com/jsonurl/specification/


# 2. Grammar
[RFC8259][1] describes the JSON data model, which includes objects, arrays,
and value literals. This document defines a new grammar for the JSON data
model called JSON&#x2192;URL. It also borrows heavily from RFC8259 so
it's recommended that you read it first.

JSON&#x2192;URL text is a sequence of characters as defined by
[RFC3986, section 2][9]. Encoded octets MUST be UTF-8 encoded UNICODE
codepoints. JSON&#x2192;URL text complies with [RFC3986, section 3.4][7] and
is suitable for use as a URI query string.

Like JSON text, JSON&#x2192;URL text is also a sequence of tokens. The set of
tokens includes structural characters, strings, numbers, and three literal
names.

	JSON-URL-text = value

These are the four structural characters:

	begin-composite    = %x28  ; ( left paren
	end-composite      = %x29  ; ) right paren
	name-separator     = %x3A  ; : colon
	value-separator    = %x2C  ; , comma

Note that while RFC8259 defines one set of tokens for arrays and another set
for objects JSON&#x2192;URL defines only a single set for both arrays and
objects. This change is due to the limitations imposed by the `query`
production defined in [RFC3986][5]. With one exception arrays and objects are
semantically no different here than in RFC8259 -- it's simply the grammar
that's different. A lookahead token will allow a JSON&#x2192;URL parser to
distinguish between arrays and objects in all cases except when the object or
array has no members.

Whitespace MUST NOT be used.

## 2.1 Values
A JSON&#x2192;URL value MUST be a composite, number, string, or one of three
literal names. The three literals and the number production are defined
exactly as in RFC8259. Object, array, and string are defined in the
following sections of this document.

	value = false / null / true / composite / number / string
	
## 2.2 Composites
A composite value is an array, object, or the `empty-composite` value
which represents an array/object with no members. In JSON&#x2192;URL, an
empty array and empty object are indistinguishable. This constitutes the
only difference between the data model defined in this document and
RFC8259.

	composite       = empty-composite / object / array
	empty-composite = begin-composite end-composite ; ()

## 2.3 Objects
An object structure is represented as a pair of parentheses surrounding one
or more name/value pairs (or members). A name is a string. A single colon
comes after each name, separating the name from the value. A single comma
separates a value from a following name.

	object = begin-composite member *( value-separator member ) end-composite
	member = string name-separator value

## 2.4 Arrays
An array structure is represented as a pair of parentheses surrounding one
or more values. Multiple values are separated by commas.

	array = begin-composite value *( value-separator value ) end-composite

As in RFC8259 there is no requirement that the values in an array be of the
same type.

## 2.5 Strings
Though semantically equivalent to `string` as defined in RFC8259
the grammar for a JSON&#x2192;URL string is quite different. A JSON&#x2192;URL
string MAY be surrounded by single-quotes (a.k.a. apostrophes), however, it is
only necessary to do so when the value would be otherwise ambiguous. In
practice, this means single quotes MUST be used to represent a string literal
'true', 'false', 'null', or number. Otherwise, their use is OPTIONAL. Object
keys are always assumed to be strings and need not be quoted even if they
would otherwise be interpreted as a Number, Boolean, or `null`.

When used in a string literal a `plus` character (U+002B) represents a single
space character (U+0020) just like the [x-www-form-urlencoded][6] type.

All text must comply with the `query` production defined in [RFC3986][7],
section 3.4. Therefore, any characters outside the set of allowed literal
characters MUST be [percent encoded][8].

There is a meaningful difference between a structural character and an encoded
structural character. When encoded, a parser MUST interpret the character
as part of a string literal. When not encoded the character retains its
structural meaning. Quoted strings need not encode structural characters.

    string        = uchar *(uchar / apos) ; unquoted string
                  / apos *qchar apos      ; quoted string
    
    uchar         = unencoded / pct-encoded / space-encoded
    
    qchar         = uchar / struct-char
    
    unencoded     = digit
                  / %x41-5A            ; A-Z  uppercase letters
                  / %x61-7A            ; a-z  lowercase letters
                  / %x2D               ; -    dash
                  / %x2E               ; .    period
                  / %x5F               ; _    underscore
                  / %x7E               ; ~    tilde
                  / %x21               ; !    exclamation point
                  / %x24               ; $    dollar sign
                  / %x2A               ; *    asterisk
                  / %x2F               ; /    solidus
                  / %x3B               ; ;    semicolon
                  / %x3F               ; ?    question mark
                  / %x40               ; @    at sign
                  
    apos          = %x27               ; '    single quote/apostrophe
    
    struct-char   = %x28               ; (    open paren
                  / %x29               ; (    close paren
                  / %x2C               ; ,    comma
                  / %x3A               ; :    colon
	
    pct-encoded   = %x25 hexdig hexdig ; %XX  percent encoded
	
    space-encoded = %x2B               ; +    plus sign
    
    hexdig        = digit / %x41-46    ; A-F  hexadecimal digits
    digit         = %x30-39            ; 0-9  digits

## 2.6 Numbers
Numbers are represented exactly as defined in RFC8259, Section 6. Note that
when used in a number the `plus` character (U+002B) is literal and MUST NOT
be interpreted by a JSON&#x2192;URL parser as a space character (as it would be
in a string literal).

## 2.7 Whitespace
The grammar defined in RFC8259 allows for "insignificant whitespace" as it
can make it easier for the human eye to parse JSON text. However,
unescaped whitespace is not allowed in a URL and escaped whitespace would
likely make it more difficult for the human eye to parse.  Therefore,
unescaped whitespace MUST NOT be present in JSON&#x2192;URL text. Escaped
whitespace MAY be present, however, it is always considered significant.

## 2.8 x-www-form-urlencoded
JSON&#x2192;URL text is designed to play well with [x-www-form-urlencoded][6]
data. JSON&#x2192;URL text MUST percent-encode literal `&` and `=` characters.
This allows one or more traditional HTML form variables to be standalone
JSON&#x2192;URL text.

## 2.9 Optional Syntaxes
A JSON&#x2192;URL parser implementation MAY support additional syntax options.
Implementations SHOULD default to the grammar described above and only allow an
alternate syntax when explicitly enabled.

### 2.9.1 Implied Arrays
If both a sender and its receiver agree a priori that the top-level value is an
array then a parser MAY accept JSON&#x2192;URL text that omits the first
`begin-composite` and last `end-composite` characters.

    implied-array = [value] *( value-separator value )

Note that, unlike an `array`, an `implied-array` may be empty (i.e. contain
zero values). There is no ambiguity in this case and the parse result MAY be an
array rather than the `empty-composite`.

`implied-array` is OPTIONAL. A JSON&#x2192;URL parser is not required to
support it.

### 2.9.2 Implied Objects
If both a sender and its receiver agree a priori that the top-level value is an
object then a parser MAY accept JSON&#x2192;URL text that omits the first
`begin-composite` and last `end-composite` characters.

    implied-object = [member] *( value-separator member )

Note that, unlike an `object`, an `implied-object` may be empty (i.e. contain
zero members). There is no ambiguity in this case and the parse result MAY be an
object rather than the `empty-composite`.

`implied-object` is OPTIONAL. A JSON&#x2192;URL parser is not required to
support it.

### 2.9.3 x-www-form-urlencoded Arrays and Objects
A parser MAY accept x-www-form-urlencoded style separators as structural
characters for a top-level array or object.

    wfu-name-separator      = %x3D  ; = equal 
    wfu-value-separator     = %x26  ; & ampersand
    
    wfu-composite           = empty-composite / wfu-object / wfu-array

    wfu-object = begin-composite wfu-member *( wfu-value-separator wfu-member ) end-composite
    wfu-member = string wfu-name-separator value
    
    wfu-array  = begin-composite value *( wfu-value-separator value ) end-composite

This may be combined with an implied array or object to form a URL query
string that is a syntactically valid, traditional HTML form data query
string.

    wfu-implied-composite = wfu-implied-object / wfu-implied-array

    wfu-implied-object    = [wfu-member] *( wfu-value-separator wfu-member )
    
    wfu-implied-array     = [value] *( wfu-value-separator value )

Note that `wfu-composite` and `wfu-implied-composite` indirectly reference
`qchar`, which allows unencoded literal `,` and `:` characters but requires
literal `&` and `=` characters to be encoded. This is intentional, and allows
JSON&#x2192;URL text to meet the goal outlined in section 2.7.  

`wfu-composite` and `wfu-implied-composite` are OPTIONAL. A JSON&#x2192;URL
parser is not required to support them.

# 3. Examples
Here are a few examples.
## 3.1 String Literal
Here are some string literals:

    word
    two+words
    Hello%2C+World!
    'Hello,+World!'
    'true'
    '42'

## 3.2 Number Literal
Here are some number literals:

    0
    1.0
    1e2
    -3e4
    42

## 3.3 Object
Here are some objects:

    (key:value)
    (Hello:World!)
    (key:value,nested:(key:value))

## 3.4 Array
Here are some arrays:

    (1)
    (1,2,3)
    (a,b,c)
    (a,b,(nested,array))
    (array,of,objects,(object:1),(object:2))
    
## 3.5 Implied Array
Here are some implied arrays:

    1
    1,2,3
    a,b,c
    a,b,(nested,array)
    array,with,objects,(object:1),(object:2)

## 3.6 Implied Object
Here are some implied objects:

    key:value
    Hello:World!
    key:value,nested:(key:value)

## 3.7 x-www-form-urlencoded Implied Array
Here are some implied arrays that make use of x-www-form-urlencoded style
separators:

    1
    1&2&3
    a&b&c
    a&b&(nested,array)
    array&with&objects&(object:1)&(object:2)
    
## 3.8 x-www-form-urlencoded Implied Object
Here are some implied objects that make use of x-www-form-urlencoded style
separators:

    key=value
    Hello=World!
    key=value&nested=(key:value)

[1]: https://tools.ietf.org/html/rfc8259        "RFC8259"
[2]: https://tools.ietf.org/html/rfc5234        "RFC5234"
[3]: https://tools.ietf.org/html/rfc2119        "RFC2119"
[4]: https://tools.ietf.org/html/rfc3986        "RFC3986"
[5]: https://tools.ietf.org/html/rfc3986#appendix-A "RFC3986, Appendix A."
[6]: https://en.wikipedia.org/wiki/Percent-encoding#The_application.2Fx-www-form-urlencoded_type "application/x-www-form-urlencoded"
[7]: https://tools.ietf.org/html/rfc3986#section-3.4 "RFC3986, Section 3.4."
[8]: https://tools.ietf.org/html/rfc3986#section-2.1 "RFC3986, Section 2.1."
[9]: https://tools.ietf.org/html/rfc3986#section-2 "RFC3986, Section 2."
[10]: https://tools.ietf.org/html/rfc1738        "RFC1738"



