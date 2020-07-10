# Container Types

The Universal Binary JSON Specification defines a total of **2 container types** matching [JSON's container types](http://json.org/):

1.  [Array Type](#array-type)
2.  [Object Type](#object-type)

Ignoring special-case optimizations, the design of the Universal Binary JSON containers is intentionally identical to JSON (the same start/end markers) and are **streaming-friendly**; more specifically they can be written out on-demand without knowing the size of the container ahead of time.

### Optimized Format

Both _array_ and _object_ container types in UBJSON support being represented in a more optimized format that can increase parsing performance as well as shrink data size in most cases (without compression). Please see [Optimized Format](#optimized-format) below for details on how to leverage this support.

# <a name="array"></a>Array Type

* * *

*   [Usage](#array-use)
*   [Example](#array-example)

The _array_ type in Universal Binary JSON is defined as:

<table id="type-ref-table" style="width: 100%;" border="0">

<thead>

<tr>

<td>Type</td>

<td>Size</td>

<td>Marker</td>

<td>Length</td>

<td>Data Payload</td>

</tr>

</thead>

<tbody>

<tr>

<td>array</td>

<td>2+ bytes**</td>

<td>[ and ]</td>

<td>Optional</td>

<td>Yes (if non-empty)</td>

</tr>

</tbody>

</table>

<sup>** See [Optimized Format](#optimized-format) below.</sup>

### <a name="array-use"></a>Usage

The _array_ type in Universal Binary JSON is equivalent to the **array** type from the [JSON specification](http://json.org/).

### <a name="array-example"></a>Example

JSON snippet (42 bytes compacted):

```json
[
    null,
    true,
    false,
    4782345193,
    153.132,
    "ham"
]
```

UBJSON snippet (21 bytes, **50% smaller**):

```
[[]
    [Z]
    [T]
    [F]
    [l][4782345193]
    [d][153.132]
    [S][i][3][ham]
[]]
```

> ✓ Universal Binary JSON format is **50% smaller** than the compacted JSON.

# <a name="object"></a>Object Type

* * *

*   [Usage](#object-use)
*   [Example](#object-example)

The _object_ type in Universal Binary JSON is defined as:

<table id="type-ref-table" style="width: 100%;" border="0">

<thead>

<tr>

<td>Type</td>

<td>Size</td>

<td>Marker</td>

<td>Length</td>

<td>Data Payload</td>

</tr>

</thead>

<tbody>

<tr>

<td>object</td>

<td>2+ bytes**</td>

<td>{ and }</td>

<td>Optional</td>

<td>Yes (if non-empty)</td>

</tr>

</tbody>

</table>

<sup>** See [Optimized Format](#optimized-format) below.</sup>

### <a name="object-use"></a>Usage

The _object_ type in Universal Binary JSON is equivalent to the **object** type from the [JSON specification](http://json.org/).

### <a name="object-example"></a>Example

JSON snippet (90 bytes compacted):

```json
{
    "post": {
        "id": 1137,
        "author": "rkalla",
        "timestamp": 1364482090592,
        "body": "I totally agree!"
    }
}
```

UBJSON snippet (82 bytes, **9% smaller**):

```
[{]
    [i][4][post][{]
        [i][2][id][I][1137]
        [i][6][author][S][i][5][rkalla]
        [i][9][timestamp][L][1364482090592]
        [i][4][body][S][i][16][I totally agree!]
    [}]
[}]
```

> ⓘ **NOTE**: The [S] (_string_) marker is omitted from each of the _names_ in the _name/value_ pairings inside the object. The JSON specification does not allow non-_string_ _name_ values, therefore the [S] marker is redundant and <span style="text-decoration: underline;">**must not**</span> be used.

# <a name="optimized-format"></a>Optimized Format

* * *

*   [Array Example](#optimized-format-example-array)
*   [Object Example](#optimized-format-example-object)
*   [Special Cases](#optimized-special-cases) (Null and Boolean)
*   [Size & Performance Benefits](#optimized-size-perf-benefits)
*   [Binary Data Support](#optimized-binary-support)

While the basic specification for the _array_ and _object_ types are identical to the JSON specification (i.e. simple beginning and end markers), both containers support _optional_ parameters that can help optimize the container for better parsing performance and smaller size. At a very high level, the optimized format for both _array_ and _object_ container types are built around two optional parameters: **type** and **count**

<table id="type-ref-table" style="width: 100%;" border="0">

<thead>

<tr>

<td>Type</td>

<td>Size</td>

<td>Marker</td>

<td>Arg. Type</td>

<td>Example</td>

<td>Desc</td>

</tr>

</thead>

<tbody>

<tr>

<td>type</td>

<td>1-byte</td>

<td>$</td>

<td>[Value Type](value-types) or [Container Type](type-referencecontainer-types/) Marker</td>

<td>[$][S]</td>

<td>string type</td>

</tr>

<tr>

<td>count</td>

<td>1-byte</td>

<td>#</td>

<td>Integer [Numeric Value](value-types#numeric-types)</td>

<td>[#][i][64]</td>

<td>count of 64</td>

</tr>

</tbody>

</table>

The effect on the container when specifying one or both parameters is as follows:

*   **type** [**<span style="color: #0000ff;">$</span>**] - when a **type** is specified, all _value_ types stored in the container (either _array_ or _object_) are considered to be of that singular type and as a result, type markers are omitted for each value in the container. This can be thought of providing the ability to create a strongly typed container in UBJSON.
    *   If a **type** is specified, it must be done so before a **count**.
    *   If a **type** is specified, a **count** must be specified as well (otherwise it is impossible to tell when a container is ending; e.g., did you just parse ']' or the int8 value of 93?)
*   **count** [**<span style="color: #0000ff;">#</span>**] - when a **count** is specified, the parser is able to know ahead of time how many child elements will be parsed. This allows the parser to pre-size any internal construct used for parsing, verify that the promised number of child _values_ were found and avoid scanning for any terminating bytes while parsing.
    *   A **count** can be specified without a **type**.

> ⓘ **NOTE**: Yes it is possible for an _array_ or _object_ to define their **type** as '[' or '{' to signal that they themselves contain additional containers!

> ⬇ **BONUS**: Parsers can provide _highly-optimized_ implementations for strongly typed containers of non-variable-length types (e.g. numeric, boolean, etc.) because the exact byte-length of the data is known![/box] Some rules that generators and parsers need to be aware of when dealing with these optional parameters is as follows:

*   [count] A **count** must be >= 0.
*   [count] A **count** <span style="text-decoration: underline; color: #339966;">can</span> be specified by itself.
*   [count] If a **count** is specified the container <span style="text-decoration: underline; color: #ff0000;">must not</span> specify an end-marker.
*   [count] A container that specifies a **count** <span style="text-decoration: underline; color: #339966;">must</span> contain the specified number of child elements.
*   [type] If a **type** is specified, it must be done so before **count**.
*   [type] If a **type** is specified, a **count** <span style="text-decoration: underline; color: #339966;">must</span> also be specified. A **type** cannot be specified by itself.
*   [type] A container that specifies a **type** <span style="text-decoration: underline; color: #ff0000;">must not</span> contain any additional type markers for any contained value.
*   [type] The **type** <span style="text-decoration: underline; color: #ff0000;">cannot</span> be No-op. Indeed, creating a container whose type is “nothing” (which is what No-op actually is) does not really mean anything.

<a name="optimized-format-example-array"></a>

## Array Example

Below are examples of incrementally more optimized representations of an _array_ in UBJSON.

### No Optimization

```
[[]
    [d][29.97]
    [d][31.13]
    [d][67.0]
    [d][2.113]
    [d][23.888]
[]]
```

### Optimized with count

```
[[][#][i][5] // An array of 5 elements.
    [d][29.97]
    [d][31.13]
    [d][67.0]
    [d][2.113]
    [d][23.8889]
// No end marker since a count was specified.
```

### Optimized with type & count

```
[[][$][d][#][i][5] // An array of 5 float32 elements.
    [29.97] // Value type is known, so type markers are omitted.
    [31.13]
    [67.0]
    [2.113]
    [23.8889]
// No end marker since a count was specified.
```

<a name="optimized-format-example-object"></a>

## Object Example

Below are examples of incrementally more optimized representations of an _object_ in UBJSON. 

> ⓘ Remember, in UBJSON the _string_ markers ([S]) are omitted from the _names_ in the _name-value_ pairs of an Object because JSON only allows _names_ of type _string_.[/box]

### No Optimization

```
[{]
    [i][3][lat][d][29.976]
    [i][4][long][d][31.131]
    [i][3][alt][d][67.0]
[}]
```

### Optimized with count

```
[{][#][i][3] // An object of 3 name:value pairs.
    [i][3][lat][d][29.976]
    [i][4][long][d][31.131]
    [i][3][alt][d][67.0]
// No end marker since a count was specified.
```

### Optimized with type & count

```
[{][$][d][#][i][3] // An object of 3 name:float32-value pairs.
    [i][3][lat][29.976] // Value type is known, so type markers are omitted.
    [i][4][long][31.131] 
    [i][3][alt][67.0] 
// No end marker since a count was specified.
```

<a name="optimized-special-cases"></a>

## Special Cases (Null and Boolean)

Up until now all the examples of leveraging **type** and **count** have illustrated the benefit of optimizing out the markers from [value types](value-types) that have a data payload (e.g. numeric values, strings, etc.); since the type of all the values are known, the markers are easily omitted. There are, however, a few special value types that have **no data payload** and the markers themselves represent the value, specifically: [_null_](value-types#null-value) and [boolean](value-types#boolean-types) (no-op is not a valid type for a container). This section will take a look at how those types behave when used with strongly-typed containers. At a high level, placing these values in a strongly-typed container provides the basic behavior of essentially pre-defining the value for every element in the container. In the case of and _array_, all the values contained in it. In the case of an _object_, all the _values_ associated with all the _names_ in the _name-value_ pairs.

### Array

```
[[][$][F][#][I][512] // 512 'false' values.
```

The example above is a strongly typed _array_ of **type** _false_ and with a **count** of 512. This simple declaration is equivalent to a **514-byte** _array_ containing 512 [F] markers; instead this single line is **6-bytes** providing a <span style="color: #339966;">**99% size reduction**</span>. Admittedly this is a selective example of leveraging this feature, but the point is that there are potentially very large performance and size optimizations available if your data can take advantage of this shorthand. 

> ⓘ Strongly-typed arrays of [_null_](value-types#null) and [boolean](value-types#boolean) **must** have an empty body. The header itself defines the container's contents.

### Object

```
[{][$][Z][#][i][3]
    [i][4][name] // name only, no value specified.
    [i][8][password]
    [i][5][email]
```

The example above is a strongly typed _object_ of **type** _null_ and with a **count** of 3. When used in the context of an _object_, specifying one of these special-case values as a **type** has the effect of setting the default _value_ for every _name-value_ pair in the object; therefore the _object_ only contains the _names_ of all the pairs. In the case of _object_s the space-savings is typically a little less drastic than in the _array_ case depending on the size of the _names_; in the case of small _names_, it could be significant, approaching a <span style="color: #339966;">**50% reduction**</span>. 

> ⓘ Strongly-typed objects of [_null_](value-types#null) and [boolean](value-types#boolean) **must not** have any _values_ specified in the body, just the _name_ portions of the _name-value_ pairs. The header itself defines the _value_ for every _name-value_ pair.
<a name="optimized-size-perf-benefits"></a>

## Size & Performance Benefits

*   [<span style="line-height: 13px;">Optimized for Parsing</span>](#optimized-size-perf-benefits-parsing)
*   [Simple Validation Mechanism](#optimized-size-perf-benefits-validation)
*   [Reduce Size up to 50%](#optimized-size-perf-benefits-size)

The benefits realized by leveraging the optimized container types in UBJSON depend heavily on the data being stored and the implementation of the generator or parser. Baring the frustration of "_it depends_" as an answer, the benefits can be viewed at a very high level as the following:<a name="optimized-size-perf-benefits-parsing"></a>

### Optimized for Parsing

By specifying a **count**, you are hinting to the parser about the number of elements to expect. The performance gains are primarily around allowing the parser to pre-size its internal data structures to exactly the right size to hold pointers to the parsed values. By specifying a **type** and **count**, the parser not only knows how many child elements to expect, as well as less data to parse and less conditions to run (no marker checks), but in the cases of fixed-length values, the parser knows the exact **byte length** of the payload! For example, consider:

```
[[][$][l][#][I][1024] // 1,024 int32 values
    [32]
    [2147483647]
    [101231]
    [77832823]
    ... 1,000 more int32 values ...
```

After the parser parses the container's header, it knows the byte length of the entire payload is 4096 and in a single read operation can read all the values in and quickly break them up into their _[int32](#numeric-types)_ representations. When you are able to leverage the **type** and **count** together to help the parser understand the payload in more detail is where the real performance gains come from.<a name="optimized-size-perf-benefits-validation"></a>

### Simple Validation Mechanism

By specifying a **count** parameter, you are telling the parser the number of child elements it should find in the container. In the case where the parser is unable to find the specified number of child elements it can quickly report a format error to the caller. This is a very simple version of verification and not as robust as say a checksum-based approach, but it still provides benefit in addition to a performance gain.<a name="optimized-size-perf-benefits-size"></a>

### Reduce Size up to 50%

This is a 1-byte-per-value reduction in any container where strong typing is used. In the case of containers holding large amounts of fairly compact data (small numbers, chars, small strings or value-types like _null_), removing the type marker from the beginning of **each** of the values in the container can almost cut the size requirements for the data in half. The smaller the containers and bigger the individual values are (large numbers, large strings) the less **size benefit** this optimization will have, but it still provides a potentially significant opportunity to the parser to optimize it's code paths for parsing large chunks of same-type _values_ (and not needing to worry about type changes mid-container). This is covered in more detail in the previous section: _Optimized for Parsing_<a name="optimized-binary-support"></a>

## Binary Data Support

This section is here for referential convenience; please see [Binary Data](type-referencebinary-data/) for information on storing binary data in UBJSON.