# Thanks

Universal Binary JSON was originally motivated by a desire to provide an on-disk & over-the-wire format that required no parsing or marshalling in CouchDB ([inspiration](https://issues.apache.org/jira/browse/COUCHDB-702)). In its [original draft form](https://docs.google.com/document/d/12SimAfBVcl8Fd-lr_SSBkM5B_PyEhDRfhgu1Lzvfpfw/edit?hl=en_US), UBJSON was much too simple of a spec with too many holes but over the next number of years and **only** with the help of the following people (among many others) did the spec grow up. I want to express my personal thanks to each one of you for all the help you lent at the different stages of UBJSON's development (and continue to provide in some cases). Sincerely, Riyad Kalla

* * *

[**Adil Baig**](http://thoughtsimproved.wordpress.com/)

Adil has been very involved in the in-depth and multi-year long discussions surrounding a more optimized container specification as well as binary data support. Adil also provided a very compelling, diff-typing proposal for an optimized container format that provided a lot of good guidance around elegant alternatives to consider.

**[Alex Blewitt](http://twitter.com/#!/alblue)**

Helped catch a number of specification errors around UTF-8 encoding in the original draft of the specification that would have been confusing/nasty to release. He also provided great feedback about the size and performance metrics for the specification.

**[Alexander Shorin](http://code.google.com/p/simpleubjson/)**

Alex is both the author of the [UBJSON Python library](https://code.google.com/p/simpleubjson/) and a valued collaborator on the Universal Binary JSON spec as it matured. Alex provided instrumental insight into the modifications made between Draft 8 and Draft 9 of the spec to help simplify the spec by removing all the duplicate (_compact_) type representations, simplifying the length-arguments for _STRING_ and _HUGE_ as well as being the one to point out that the _length_ arguments for the _ARRAY_ and _OBJECT_ container types are effectively useless once the streaming-format support was added (and do not make generator code or parsing code any easier or more performant).

[**Bjørn Reese**](https://github.com/breese)

Bjørn has been involved in most all of the _binary data support_ discussions that have taken place since 2012\. His detail-oriented contributions helped move the discussion forwad.

**[John Cowan](http://tech.groups.yahoo.com/group/json/message/1734)**

John was the one that recommended using UTF-8 string-encoded values (or _huge_) for arbitrarily huge numbers after seeing my desire to avoid including any non-portable constructs into the binary format.

Given that the discussion on numeric formats had been a very active one with lots of feelings on all sides, it was a boon to have John step up with such a simple suggestion that allowed for maximum compatibility and portability. It was a win-win all the way around.

**[Michael Makarenko](http://www.m1xa.com/)** (aka "M1xA")

Michael is the author behind the [Ubjson.NET library](libraries) and contributor of the _int16_ and _float_ numeric types to the specification. For numeric-heavy (e.g. scientific) data, the inclusions of the in16 and float types can lead to significant space savings when writing out values in the Universal Binary JSON format.

Michael has also gone to great lengths to make the .NET implementation of UBJSON as tight and performant as possible; collaborating on benchmark design and testing data as well as compatibility testing between implementations to ensure a great Universal Binary JSON experience for .NET developers.

In addition to development, Michael has helped contribute to the growth of the Universal Binary JSON community with [articles about the specification](http://habrahabr.ru/blogs/open_source/130112/).

**[Paul Davis](http://davispj.com/)**

While approaching the CouchDB team for feedback on the Universal Binary JSON spec, I met Paul who was willing to spend a significant amount of time reviewing the specification and recommending suggestions, changes and improvements from everything the CouchDB team has learned by dealing closely with JSON for years.

Paul pointed out the shortcomings of prefixing the length to the two container types if the specification could ever be used easily with services or apps that streamed UBJ format for huge runs of data that the server couldn't load, buffer and count ahead of time before responding to the client. In order to more easily support streaming, unknown-length container types had to be added.

Paul also pointed out the importance of a NO_OP/SKIP/IGNORE type that can be useful during a long-lived streaming operation where the server may be waiting on something (like a DB) and you need to keep the connection alive between client/server and avoid the client timing out, but you need the client to know the data it is receiving is just meant as a "Hang on" message from the server and not actual data. This is where the NO_OP command comes in handy.

**[Stephan Beal](http://tech.groups.yahoo.com/group/json/message/1686)**

Stephan helped quite a bit with understanding the implications of a >= 64-bit numeric format and the implications of portability across a number of popular platforms.

* * *

**[JSON Specification Group](http://tech.groups.yahoo.com/group/json/)**

I would like to personally thank everyone in the JSON Specification Group. The amount of feedback and help with the specification has been wonderful, constructive and creative. It also lead to one of the busiest conversations in the last year!