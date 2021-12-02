# Binary Data

_Page not organized well and under development, but here are the highlights_...

## Overview

Support for binary data in the Universal Binary JSON specification was in discussion for 2 years before it was finalized. Many, many different approaches were considered and discarded all in the name of maintaining compatibility with JSON while keeping an eye on performance. The result is a surprisingly simple and binary-efficient construct that is also easily translated to JSON and back to UBJSON again with the help of a good library, namely: a [strongly-typed](container-types#optimized-format) _array_ of [_uint8_](value-types#numeric-types) values.

## Compatibility with JSON

Representing binary data efficiently in Universal Binary JSON while still maintaining compatibility with JSON is deceptively simple: leverage a [strongly-typed _array_](container-types#optimized-format) of [_uint8_](value-types#numeric-types) values -- essentially a list of integers. There is no explicit _binary_ [type](type-reference), but instead the ability to represent binary inside of Universal Binary JSON in a very optimized and JSON-compatible construct. The [#1 goal](http://ubjson.org/#goals) of Universal Binary JSON is compatibility with JSON. Compatibility is defined as:

```
if 
    A.ubjson -> translated to -> B.json
    &&
    B.json -> translated to -> C.ubjson
then
    A.ubjson == C.ubjson
```    

All of the Universal Binary JSON value and container types are 1:1 compatible with JSON. The only _semantically_ (but not _structurally_) incompatible construct in UBJSON is strongly-typed containers in that once the container is converted to JSON the typing of the container is lost. Converting the container back to UBJSON and re-enabling the strong-typing <span style="text-decoration: underline;">does require assistance</span> from the encoding library. Since JSON has no direct support for binary data or this style of strongly-typed container, the translation to JSON converts the strongly-typed _array_ to an _array_ of simple JSON types - in the case of binary data, it would be an _array_ of _number_ values (In the example above this is the translation step from A.ubjson to B.json). Going from JSON back to UBJSON (B.json -> C.ubjson) has the potential for losing the strongly-typed container information and has to be handled with care to re-enable the optimized representation of that information back in the UBJSON format.

## Library Implementation Recommendation

The library implementors are encouraged to provide this functionality in the form of two _optional settings_ that can be turned on during generation:

*   <span style="line-height: 13px;">[x] Automatically use strongly typed containers when possible</span>
*   [x] Force use of strongly typed containers based on first element type

> ⓘ Specific naming and implementation is up to the developer. This is merely a suggestion on how to handle this situation as elegantly as possible for the client.

The idea being that the library can either make an automated attempt at reconstructing the strongly typed containers OR if you have a lot of knowledge of your data, you can force the library to reconstitute what looks to be a strongly typed container based on the fist element type. 

> ⚠ If _Force_ is used the library should take care to detect and fail if a different type of value is found in the container during generation. More specifically, the library should remember the first element type and continue checking types as it is generating UBJSON to ensure the type continues to stay consistent.[/box] _Still under development..._

## Performance Considerations

Something to be aware of when converting UBJSON containing a large amount of binary data is that each strongly-typed container of _uint8_ values will convert to a JSON array of _number_ values, because this translation also introduces a ',' character between every value in the array, this effectively **doubles the size** of the binary data.