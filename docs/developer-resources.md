# Developer Resources

This page contains information for developers looking to develop a Universal Binary JSON library.

*   [Library Implementation Requirements](#library_req)
*   [Best Practices](#best_practice)
*   [Example Files](#example_files)

# <a name="library_req"></a>Library Implementation Requirements

Libraries implementing the Universal Binary JSON spec must adhere to the following guidelines:

*   Parsers must follow a "writer-makes-right" policy - more specifically, if a parser encounters unexpected or invalid data (e.g. negative container length value) an exception should be thrown and parsing stopped.

# <a name="best_practice"></a>Best Practices

*   [Optimizing Container Performance](#best_container_perf)
*   [Using Smallest Number Representation](#best_smallest_num)
*   [Handling High-Precision Numbers on Unsupported Platforms](#best_high_prec_num)

Through work with the community, feedback from others and our own experience with the specification, below are some of the best-practices collected into one place making it easy for folks working with the format to find answers to the more flexible portions of the spec.

## <a name="best_container_perf"></a>Optimizing Container Performance

> ✓ **Why:** (Potentially large) data size reduction and parsing performance increase. **How**: Homogeneous data type in a container.

Very large performance advantages are available when writing out _ARRAY_ or _OBJECT_ containers that contain same-type values. Be sure to read through the [_optimized container format_](container-types#optimized-format) that can be leveraged in these cases. A typical level of optimization is being able to omit all the marker characters for all same-typed values in a container, making the sizes of all typical [_value types_](type-reference) 1-byte smaller. An a-typical level of optimization, that leads to the biggest reduction, is for all [1-byte value types](type-reference) (e.g. _NO-OP_, _NULL_, etc); when used in conjunction with the [_optimized container format_,](container-types#optimized-format) the values themselves can be omitted from the container entirely leading to a space savings that approaches 100% as the size of the container grows.

## <a name="best_smallest_num"></a>Using Smallest Number Representation

> ✓ **Why:** [~50% size reduction](#size) for numbers > 5 digits and < 20 digits. **How**: Always use the most compact numeric type possible when writing UBJSON.

Numeric values can be represented in [a number of ways](value-types#numeric-types) in UBJSON; you can reduce the size of your UBJSON by inspecting the stored value and ensuring it is represented in the most-compact numeric representation possible when storing the UBJSON blob. Keep in mind that varying the type of values inside of a container may impact your ability to use the **type** parameter to [optimize container storage](container-types#optimized-format).

## <a name="best_high_prec_num"></a>Handling High-Precision Numbers on Unsupported Platforms

> ✓ **Why:** Cleanly handle > 64-bit numbers on platforms that don't support them. **How**: By using the _[high-precision](value-types#numeric-types-gt-64bit)_ type.

Not every language supports arbitrarily long numbers and some not even numbers greater than 64-bits in size. In order to safely allow the transport and handling of > 64-bit numbers across every platform, UBJSON provides the _high-precision_ numeric type. The _high-precision_ type is a string-based type (identical in format to the [_string_](value-types#string-type) type) that provides a universally compatible mechanism by which arbitrarily large or precise numbers can be handled. For platforms with arbitrarily large/precise number support, they are free to parse the _high-precision_ value into a native type; for platforms without support, the _high-precision_ value can be safely passed on, persisted to storage or handled in other non-numeric ways while still allowing the client to handle the request and not overflow or otherwise balk at the unsupported numeric type. That said, for libraries written to support platforms that do not natively support arbitrarily large or precise values, the following guidance can be employed to provide a safe and consistent behavior when encountering them:

1.  **[Default] <span style="color: #0000ff;">Exception/Error</span>:** Throw an exception(or return an error) when an unsupported _high-precision_ value is encountered during parsing. The platform doesn't support them so allow the client a chance to be aware of the fact that it is receiving data it won't know how to parse into a native type.
2.  **[Optional] <span style="color: #0000ff;">Handle as a String</span>:** (<span style="text-decoration: underline;">must be user-enabled</span>) In the case where the client doesn't need to do any processing of the value and is just doing pass-through like persisting it to a data store, treat the _high-precision_ value as a _string_ and return it to the caller.
3.  **[Optional] <span style="color: #0000ff;">Skip</span>**: (<span style="text-decoration: underline;">must be user-enabled</span>) Provide the ability for the parser to optionally skip unsupported values during parsing. Be aware that this is a dangerous approach and <span style="color: #ff0000;">**will likely lead to data loss**</span> (skipped values won't be visible to the client), but in the case where a client _must_ be able to parse any and all UBJSON it received even if it doesn't support arbitrarily large or precise numbers, then this has to be considered.

These guidelines should provide the most functional experience for a client to work with UBJSON on their platform of choice.

# <a name="example_files"></a>Example Files

> ⚠ Example files below only support Draft 8

You can find files to test your implementation with [here](https://github.com/thebuzzmedia/universal-binary-json-java/tree/master/src/test/resources/org/ubjson). There are _formatted-json_, _compacted-json_ and _UBJ_ versions of each of the testing files contained in the repository. The simple Java classes that have matching names to the UBJ files are Java class representations of the files (for Java testing) and the _Marshaller_ classes are the hand-coded serialization and deserialization code used to write out and read in those test files from UBJ format. Even if you are not working in Java, you can use those classes as a high level guide if you are curious or ignore them completely and just test against the raw file resources.