# Specification

1.  [Quick Start](#quickstart)
2.  [License](#license)
3.  [Why](#why)
4.  [Goals](#goals)
5.  [Data Format](#data-format)
6.  [Size Requirements](#size-requirements)
7.  [Endianness](#endianness)
8.  [MIME Type](#mime-type)
9.  [File Extension](#file-extension)
10. [Requests for Enhancement (RFE)](#rfe)

# Quick Start

You know what JSON is and you understand data formats and just want the good bits?

*   Keep the [Type Reference](type-reference) open in a tab to show you the markers and type definitions all in one page.
*   Details on the [Value Types](value-types) (13 of them)
*   Details on the [Container Types](container-types) (2 of them)
    *   Don't forget containers have an optional [_optimized format_](container-types#optimized-format) you can leverage.
*   Grab a [UBJSON library](libraries) for your favorite language or platform (or write your own!)
*   Discuss questions about the Spec or Libraries in the [Google Group](https://groups.google.com/forum/?fromgroups#!forum/universal-binary-json).
*   File bugs or issues in [GitHub](https://github.com/thebuzzmedia/universal-binary-json/issues)!

# License



The Universal Binary JSON Specification is licensed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0.html). Use of the spec, either as-defined or a customized extension of it, is intended to be commercial-friendly. The ultimate purpose of this specification is to provide a useful tool for software developers to leverage in any way they see fit.

# Why



[JSON](http://json.org/) has become a ubiquitous text-based file format for data interchange. Its simplicity, ease of processing and (relatively) rich data typing made it a natural choice for many developers needing to store or shuffle data between systems quickly and easy. Unfortunately, marshalling native programming language constructs in and out of a text-based representations does have a measurable processing cost associated with it. In high-performance applications, avoiding the text-processing step of JSON can net big wins in both processing time and size reduction of stored information, which is where a binary JSON format becomes helpful. Attempts to make using JSON faster through binary specifications like [BSON](http://bsonspec.org/), [BJSON](http://bjson.org/) or [Smile](http://wiki.fasterxml.com/SmileFormatSpec) exist, but have been [rejected](https://issues.apache.org/jira/browse/COUCHDB-702) from [mass-adoption](http://bsonspec.org/#/implementation) for two reasons:

1.  **Custom (Binary-Only) Data Types**: Inclusion of custom data types that have no ancillary in the original JSON spec, leaving room for incompatibilities to exist as different implementations of the spec handle the binary-only data types differently.
2.  **Complexity**: Some specifications provide higher performance or smaller representations at the cost of a [much more complex specification](http://wiki.fasterxml.com/SmileFormatSpec), making implementations more difficult which can slow or block adoption. One of the key reasons JSON became as popular as it did was because of its ease of use.

BSON, for example, defines types for binary data, regular expressions, JavaScript code blocks and other constructs that have no equivalent data type in JSON. BJSON defines a _binary_ data type as well, again leaving the door wide open to interpretation that can potentially lead to incompatibilities between two implementations of the spec and Smile, while the closest, defines more complex data constructs and generation/parsing rules in the name of absolute space efficiency. These are not short-comings, just trade-offs the different specs made in order to service specific use-cases. The existing binary JSON specifications all define incompatibilities or complexities that undo the singular tenant that made JSON so successful: **simplicity**. JSON's simplicity made it accessible to anyone, made implementations in every language available and made explaining it to anyone consuming your data immediate. Any successful binary JSON specification must carry these properties forward for it to be genuinely helpful to the community at large. This specification is defined around a singular marker-based construct used to build up and represent JSON values and objects. Reading and writing the format is trivial, designed with the goal of being understood in under 10 minutes (likely less if you are very comfortable with JSON already). 
> ⓘ **TIP**: UBJSON is built exclusively out of marker-characters like 'C' (for CHAR), 'S' (for STRING), etc. followed by either the payload itself, or a length and then the payload... that's it!

Fortunately, while the Universal Binary JSON specification carries these tenants of simplicity forward, it is also able to take advantage of optimized binary data structures that are (on average) 30% smaller than compacted JSON and specified for ultimate read performance; bringing **simplicity, size** and **performance** all together into a single specification that is 100% compatible with JSON.

## Why not JSON+gzip?

On the surface simply gzipping your compacted JSON may seem like a valid (and smaller) alternative to using the Universal Binary JSON specification, but there are two significant costs associated with this approach that you should be aware of:

1.  At least a [50% performance overhead](http://www.cowtowncoder.com/blog/archives/2009/05/entry_263.html) for processing the data.
2.  Lack of data clarity and inability to inspect it directly.

While gzipping your JSON will give you great compression, about 75% on average, the overhead required to read/write the data becomes significantly higher. Additionally, because the binary data is now in a compressed format you can no longer open it directly in an editor and scan the human-readable portions of it easily; which can be important during debugging, testing or data verification and recovery. Utilizing the Universal Binary JSON format will typically provide [a 30% reduction in size](#size) _and_ store your data in an optimized format offering you much higher performance while still allowing you to open the file directly and read through it. If you had a usage scenario where your data is put into long-term cold storage and pulled out in large chunks for processing, you might even consider gzipping your Universal Binary JSON files, storing those, and when they are pulled out and unzipped, you can then process them with all the speed advantages of UBJSON. As always, deciding which approach is right for your project depends heavily on what you need.

# Goals



The Universal Binary JSON specification has 3 goals: **1\. Universal Compatibility**

Meaning absolute compatibility with the JSON spec itself as well as only utilizing data types that are natively supported in all popular programming languages.

This allows 1:1 transforms between standard JSON and Universal Binary JSON as well as efficient representation in all popular programming languages without requiring parser developers to account for strange data types that their language may not support.

**2\. Ease of Use**

The Universal Binary JSON specification is intentionally defined using a single core data structure to build up the entire specification.

This accomplishes two things: it allows the spec to be understood quickly and allows developers to write trivially simple code to take advantage of it or interchange data with another system utilizing it.

**3\. Speed / Efficiency**

Typically the motivation for using a binary specification over a text-based one is speed and/or efficiency, so strict attention was paid to selecting data constructs and representations that are (roughly) 30% smaller than their compacted JSON counterparts and optimized for fast parsing.

# Data Format



The Universal Binary JSON specification utilizes a single construct with two optional segments (_length_ and _data)_ for all types:

<pre>
[<b>type</b>, 1-byte char]([integer numeric <b>length</b>])([<b>data</b>])
</pre>

Each element in the tuple is defined as:

*   **type**
    *   A 1-byte ASCII char used to indicate the [type](type-reference) of the data following it.
*   **length** (_OPTIONAL_)
    *   A positive, integer [numeric type](value-types#numeric-types) (int8, uint8, int16, int32, int64) specifying the length of the following data payload.
*   **data** (_OPTIONAL_)
    *   A run of bytes representing the actual binary data for this type of value.

Some value are simple enough that just writing the 1-byte ASCII marker into the stream is enough to represent the value (e.g. _null_) while others have a _type_ that is specific enough that no _length_ is needed as the length is implied by the type (e.g. _int32_) while others still require both a _type_ and a _length_ to communicate their value (e.g. _string_).

## Types

Universal Binary JSON defines a number of [_Value Types_](value-types) and [_Container Types_](type-referencecontainer-types/) that map directly to [JSON's types](http://json.org/). For the most part the correlation is 1:1 except in the case of [_numeric_ types](value-types#numeric-types) where UBJSON defines many more specific types of number storage and representation than JSON's single _number_ type.

*   [Type Reference](type-reference) (Overview)
    *   [Value Types](value-types)
    *   [Container Types](type-referencecontainer-types/)

# Size Requirements



The Universal Binary JSON specification tries to strike the perfect balance between space savings, simplicity and performance. Data stored using the Universal Binary JSON format are on average **30% smaller** as a rule of thumb. As you can see from some of the examples in this document though, it is not uncommon to see the binary representation of some data lead to [a 50% or 60% size reduction](container-types#array-type-example) without compression. The size reduction of your data depends heavily on the type of data you are storing. It is best to do your own benchmarking with a comprehensive sampling of your own data. 
> 📄 The Universal Binary JSON specification does not use compression algorithms to achieve smaller storage sizes. The size reduction is a side effect of the efficient binary storage format.

## Size Reduction Tips

The amount of storage size reduction you'll experience with the Universal Binary JSON format will depend heavily on the type of data you are encoding. Some data shrinks considerably, some mildly and some not at all, but in every case your data will be stored in a much more efficient format that is faster to read and write. Below are pointers to give you an idea of how certain data may shrink in this format:

*   _null_**,** _true_ and _false_ values will be **75% smaller** (80% in the case of _false_)
*   Large _numeric_ values (> 5 digits < 20 digits) will be **50% smaller**.
*   _array_ and _object_ containers will be **1-byte-per-value smaller**.
*   Leveraging the [_optimized container format_](container-types#optimized-format) can lead to a **significant** size reduction in environments where container data is of the same type.
*   _string_ values are 2-10 bytes bigger _per string_ (depending on the length of the string being represented by the smaller integer numeric type).

One of the great things about the Universal Binary JSON format is that even though most all your data will be represented in a smaller footprint, you still get two big wins:

1.  A smaller data format means faster writes and smaller reads. It also means less data to process when parsing.
2.  Binary format means no encoding/decoding primitive values to text and no parsing primitive values from text.

# Endianness



The Universal Binary JSON specification requires that all numeric values be written in [Big-Endian](http://en.wikipedia.org/wiki/Endianness) order.

# MIME Type



The Universal Binary JSON specification is a binary format and recommends using the following mime type: 
```
application/ubjson
```

This was added directly to the specification in hopes of avoiding [similar confusion with JSON](http://stackoverflow.com/questions/477816/the-right-json-content-type).

# File Extension



"**ubj**" is the [recommended file extension](http://www.fileinfo.com/extension/ubj) when writing out files using the Universal Binary JSON format (e.g. "_user.ubj_"). The extension stands for "_Universal Binary JSON_" and has no known conflicting mappings to other file formats.

<a name="rfe"></a>
# Requests for Enhancement (RFE)



All (proposed) changes to the specification are being tracked in [GitHub](https://github.com/thebuzzmedia/universal-binary-json/issues).