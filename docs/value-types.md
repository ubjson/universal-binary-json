# Value Types

The Universal Binary JSON Specification defines a total of **13 value types** (to [JSON's 5 value types](http://json.org)).

The reason for the increased number of value types is because UBJSON defines **8 numeric value types** (to JSON's 1) allowing for highly optimized storage/retrieval of numeric values depending on the necessary precision; in addition to a number of other more optimized representations of JSON values.

The specifications for each of the Universal Binary JSON Specification value types are below.

1.  [Null Value](#null-value)
2.  [No-Op Value](#no-op-value)
3.  [Boolean Types](#boolean-types)
4.  [Numeric Types](#numeric-types)
5.  [Char Type](#char-type)
6.  [String Type](#string-type)
7.  [Binary Data](#binary-data)

# Null Value

* * *

*   [Usage](#null-use)
*   [Example](#null-example)

The _null_ value in Universal Binary JSON is defined as:

| Type | Size | Marker | Length | Data Payload |
| --- | --- | --- | --- | --- |
| null |  1-byte |  Z |  No |  No |


### Usage

The _null_ value in Universal Binary JSON is equivalent to the **null** value from the [JSON specification](http://json.org/).

### Example

JSON snippet:

```json
{
    "passcode": null
}
```

UBJSON snippet (using block-notation):

```
[{]
    [i][8][passcode][Z]
[}]
```

# No-Op Value

* * *

*   [Usage](#noop-use)
*   [Example](#noop-example)

The _no-op_ value in Universal Binary JSON is defined as:

| Type | Size | Marker | Length | Data Payload |
| --- | --- | --- | --- | --- |
| noop |  1-byte |  N |  No |  No |

### Usage

The intended usage of the _no-op_ value is as a valueless signal between a producer (most likely a server) and a consumer (most likely a client) to indicate activity; for example, as a **keep-alive** signal so a client knows a server is still working and hasn't hung or timed out. There is no equivalent to _no-op_ value in the original [JSON specification](http://json.org/). The _NO-OP_ value is meant to be a **valueless** value; meaning it can be added to the **elements of a container** and when parsed by the receiver, the _no-op_ values are simply skipped and carry know meaningful value with them. For example, the two following _array_ elements are considered equal (using JSON format for readability):

```json
["foo", "bar", "baz"]
```

and

```json
["foo", no-op, "bar", no-op, no-op, no-op, "baz", no-op, no-op]
```

There are a number of interesting advantages to having a valueless-value defined directly in the spec.

### Example

Consider a web service that performs an expensive operation that can take quite a while (let's say 5 minutes):

```
<start response>
[N]
<10 second delay>
[N]
<10 second delay>
[N]
<10 second delay>
<...receiving data...>
<10 second delay>
[N]
<10 second delay>
[N]
<...receiving remainder of data...>
<end response>
```

Most clients by default will timeout after 60 seconds and more aggressive clients will timeout even faster. To help let clients know that the server has not hung, is still alive and is still processing the request the server can reply at some determined interval (e.g. every X seconds) with the _no-op_ value and the client can parse it, acknowledge it and reset its timeout-disconnect timer as a result. 

Another example of leveraging _no-op_ in an interesting way is modeling an efficient **delete** operation for UBJSON on-disk when elements of a container are removed. Instead of reading the entire container, removing the elements and writing the whole thing out again, _no-op_ bytes can simply be written over the records that were removed from the containers. When the record is parsed, it is semantically identical to a container without the values. 

These are just a few examples of how you can leverage the _no-op_ value.

# Boolean Types

* * *

*   [Usage](#boolean-use)
*   [Example](#boolean-example)

The _boolean_ types in Universal Binary JSON are defined as:

| Type | Size | Marker | Length | Data Payload |
| --- | --- | --- | --- | --- |
| true |  1-byte |  T |  No |  No |
| false |  1-byte |  F |  No |  No |

### Usage

A _boolean_ type is represented in Universal Binary JSON similar to the [JSON specification](http://json.org/): using a _T_ (true) and _F_ (false) character marker.

### Example

JSON snippet:

```json
{
    "authorized": true,
    "verified": false
}
```

UBJSON snippet (using block-notation):

```
[{]
    [i][10][authorized][T]
    [i][8][verified][F]
[}]
```

# Numeric Types

* * *

*   [Usage](#numeric-use)
*   [Example](#numeric-example)
*   [Infinity](#numeric-infinity)
*   [Signage & Min/Max Values](#numeric-sign-min-max)
*   [64-bit Values](#numeric-64bit)
*   [Larger than 64-bit Values](#numeric-gt-64bit)
*   [Byte Order / Endianness](#numeric-byte-order-endianness)
*   [Storage Size](#numeric-storage-size)

There are 8 numeric types in Universal Binary JSON and are defined as:

| Type | Size | Marker | Length | Data Payload |
| --- | --- | --- | --- | --- |
| int8 |  2-bytes |  i |  No |  Yes |
| uint8 |  2-bytes |  U |  No |  Yes |
| int16 |  3-bytes |  I |  No |  Yes |
| int32 |  5-bytes |  l |  No |  Yes |
| int64 |  9-bytes |  L |  No |  Yes |
| float32 |  5-bytes |  d |  No |  Yes |
| float64 |  9-bytes |  D |  No |  Yes |
| high-precision number |  1-byte + int num val + string byte len |  H |  Yes |  Yes (if non-empty) |

In JavaScript (and JSON) the _[Number](http://people.mozilla.org/~jorendorff/es5.html#sec-8.5)_ type can represent any numeric value, while in most other languages multiple (discrete) numeric types exist to describe different sizes and types of numeric values; this allows the runtime to handle numeric operations more efficiently. 

In order for the Universal Binary JSON specification to be a performant alternative to JSON, support for these most common numeric types had to be added to allow for more efficient reading and writing of numeric values. 

Trying to maintain a single numeric type in UBJSON would have lead to parsing complexity, requiring each language to further inspect the numeric value and marshall it down to the most appropriate internal type. By pre-defining these different numeric types directly in UBJSON, it allows for either a direct conversion into a native language type (e.g. Java) or a straight forward marshaling into the nearest-supported language type (e.g. Erlang).

### Usage

The intended usage of the different _numeric_ types are to efficiently store numbers in a space and encoding-optimized format. 

> ⓘ It is always recommended to use the smallest numeric type that fits your needs. For data with a large amount of numeric data, this can cut down the size of the payloads significantly (on average a **50% reduction** in size).

### Example

JSON Snippet:

```json
{
    "int8": 16,
    "uint8": 255,
    "int16": 32767,
    "int32": 2147483647,
    "int64": 9223372036854775807,
    "float32": 3.14,
    "float64": 113243.7863123,
    "huge1": "3.14159265358979323846",
    "huge2": "-1.93+E190",
    "huge3": "719..."
}
```

UBJSON snippets (using block-notation):

```
[i][4][int8][i][16]
[i][5][uint8][U][255]
[i][5][int16][I]32767]
[i][5][int32][l][2147483647]
[i][5][int64][L][9223372036854775807]
[i][7][float32][d][3.14]
[i][7][float64][D][113243.7863123]
[i][5][huge1][H][i][22][3.14159265358979323846]
[i][5][huge2][H][i][10][-1.93+E190]
[i][5][huge3][H][U][200][719...]
```

### Infinity

Numeric values of **infinity** are encoded as a [_null_](#null) value. (See [ECMA](http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf) and [JSON](http://json.org/json.ppt))

### Signage & Min/Max Values

The min/max range of values (_inclusive_) for each numeric type are as follows:
| Type | Signed | Min Value | Max Value |
| --- | --- | --- | --- |
| int8 |  Yes |  -128 |  127 |
| uint8 |  No |  0 |  255 |
| int16 |  Yes |  -32,768 |  32,767 |
| int32 |  Yes |  -2,147,483,648 |  2,147,483,647 |
| int64 |  Yes |  -9,223,372,036,854,775,808 |  9,223,372,036,854,775,807 |
| float32 |  Yes |  See IEEE 754 Spec |  See IEEE 754 Spec |
| float64 |  Yes |  See IEEE 754 Spec |  See IEEE 754 Spec |
| high-precision number |  Yes |  Infinite |  Infinite |

### 64-bit Integers

While almost all languages native support 64-bit integers, not all do (e.g. C89 and JavaScript ([yet](http://wiki.ecmascript.org/doku.php?id=harmony:binary_data_discussion&s=int64))) and care must be taken when encoding 64-bit integer values into binary JSON then attempting to decode it on a platform that doesn’t support it. 

If you are fully aware of the platforms and runtime environments your binary JSON is being used on and know they all support 64-bit integers, then you are fine. 

If you are trying to deserialize 64-bit integers in a client’s browser in JavaScript or another environment that does not support 64-bit integers, then you will want to take care to skip them in the input or have the client producing them encode them as _double_ or _high-precision_ values if that is easier to handle. 

Alternatively you might consider encoding your 64-bit values as doubles if you know you are going from the server to a client JavaScript environment with the binary-encoded information.

### High-Precision Numbers (Larger than 64-bit)

The _high-precision number_ type is an ultra-portable mechanism by which arbitrarily large (or precise) numbers, greater than 64-bit in size, are encoded as a UTF-8 string and passed between systems that support them. This allows _high-precision number_ values to degrade gracefully on systems that do not have a built-in type to support numeric values larger than 64-bit. Please refer to the [Best Practices](developer-resources#best_practice) page for techniques on working around the lack of larger-than-64-bit numeric types on certain platforms if you need them. 

_high-precision number_ values must be written out in accordance with the original [JSON _number_ type specification](http://json.org/).

### Byte Order / Endianness

All integer types (_int8, uint8, int16, int32_ and _int64_) are written in [most-significant-bit order](http://en.wikipedia.org/wiki/Most_significant_bit) (high byte written first, aka "[big endian](http://en.wikipedia.org/wiki/Endianness)"). 

_float32_ values are written in IEEE 754 [single precision floating point format](http://en.wikipedia.org/wiki/IEEE_754-1985), which is the following structure:

*   Bit 31 (1 bit) – sign
*   Bit 30-23 (8 bits) – exponent
*   Bit 22-0 (23 bits) – fraction (significand)

_float64_ values are written in IEEE 754 [double precision floating point format](http://en.wikipedia.org/wiki/Double_precision_floating-point_format#Double_precision_binary_floating-point_format), which is the following structure:

*   Bit 63 (1 bit) – sign
*   Bit 62-52 (11 bits) – exponent
*   Bit 51-0 (52 bits) – fraction (significand)

### Storage Size

The size of the _high-precision number_ type "on-disk" follows the same structure and sizing of the [_string_](#string-type) type (see **Storage Size** section). 

All other numeric types storage size is reflected at the beginning of this section as well as in the [Type Reference](type-reference) table.

# Char Type

* * *

*   [Usage](#string-use)
*   [Example](#string-example)
*   [Encoding](#string-encoding)
*   [Storage Size](#string-storage-size)

The _char_ type in Universal Binary JSON is defined as:

| Type | Size | Marker | Length | Data Payload |
| --- | --- | --- | --- | --- |
| char |  2-bytes |  C |  No |  Yes |

### Usage

The _char_ type in Universal Binary JSON is an unsigned byte meant to represent a single printable ASCII character (decimal values 0-127). Put another way, the _char_ type represents a single-byte UTF-8 encoded character. 

> 📄 The _char_ type is synonymous with 1-byte, UTF8 encoded value (decimal values 0-127). A _char_ value **must not** have a decimal value larger than 127. 

The _char_ type is functionally identical to the [_uint8_ type](#numeric), but semantically is meant to represent a character and not a numeric value.

### Example

JSON snippet:

```json
{
    "rolecode": "a",
    "delim": ";",
}
```

UBJSON snippet (using block-notation):

```
[[]
    [i][8][rolecode][C][a]
    [i][5][delim][C][;]
[]]
```

# String Type

* * *

*   [Usage](#string-use)
*   [Example](#string-example)
*   [Encoding](#string-encoding)
*   [Storage Size](#string-storage-size)

The _string_ type in Universal Binary JSON is defined as:

| Type | Size | Marker | Length | Data Payload |
| --- | --- | --- | --- | --- |
| string |  1-byte + int num val + string byte len |  S |  Yes |  Yes (if non-empty) |

### Usage

The _string_ type in Universal Binary JSON is equivalent to the **string** type from the [JSON specification](http://json.org/).

### Example

JSON snippet:

```json
{
    "username": "rkalla",
    "imagedata": "<huge string payload...>"
}
```

UBJSON snippet (using block-notation):

```
[[]
    [i][8][username][S][i][5][rkalla]
    [i][9][imagedata][S][l][2097152][...huge string payload...]
[]]
```

### Encoding (UTF-8)

The JSON specification does not dictate a specific required encoding, it does however use [UTF-8](http://en.wikipedia.org/wiki/UTF-8) as the default encoding. 

The Universal Binary JSON specification dictates UTF-8 as the **required string encoding** (this includes the _high-precision number_ type as it is a string-encoded value). This will allow you to easily exchange binary JSON between open systems that all support and follow this encoding requirement as well as providing a number of [advantages and optimizations](http://en.wikipedia.org/wiki/UTF-8#Advantages).

### Storage Size

The size of the _string_ type varies depending on two things:

1.  The integral numeric type used to describe the length of the string (e.g. _int8, in16, int32_ or _int64_)
2.  The UTF-8 encoded size, in bytes, of the string.

For example, English typically uses 1-byte per character, so the string “hello” has a length of 5. The same string in Russian is “привет” with a byte length of 12 and in Arabic the text becomes “مرحبا” with a byte length of 10. 

Here are some examples of what different _string_ values look like to illustrate the point:


| Binary Representation | Description |
| --- | --- |
| `[S][i][5][hello]` |  8 bytes, string UTF-8 "hello" (English) |
| `[S][i][12][привет]` |  15 bytes, string UTF-8 "hello" (Russian) |
| `[S][i][10][مرحبا]` | 13 bytes, string UTF-8 "hello" (Arabic) |
| `[S][I][1024][...1k long string...]` |  1 + 3 + 1024 bytes = 1028 bytes total |

# Binary Data

* * *

_Please see the [Binary Data](binary-data) page..._
