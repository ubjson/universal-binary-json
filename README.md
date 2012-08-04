Universal Binary JSON
=====================

Community workspace for the [Universal Binary JSON Specification][ubjson].

Introduction
============

[JSON][json] has become a ubiquitous text-based file format for
data interchange. Its simplicity, ease of processing and (relatively) rich data
typing made it a natural choice for many developers needing to store or shuffle
data between systems quickly and easy.

Unfortunately, marshaling native programming language constructs in and out of
a text-based representations does have a measurable processing cost associated
with it.

In high-performance applications, avoiding the text-processing step of JSON can
net big wins in both processing time and size reduction of stored information,
which is where a binary JSON format becomes helpful.

Why
===

Attempts to make using JSON faster through binary specifications like
[BSON][bson], [BJSON][bjson] or [Smile][smile] exist, but have been rejected
from mass-adoption for two reasons:

* Custom (Binary-Only) Data Types:
  Inclusion of custom data types that have no ancillary in the original JSON
  spec, leaving room for incompatibilities to exist as different implementations
  of the spec handle the binary-only data types differently.
* Complexity: Some specifications provide higher performance or smaller
  representations at the cost of a much more complex specification, making
  implementations more difficult which can slow or block adoption. One of the key
  reasons JSON became as popular as it did was because of its ease of use.

Goals
=====

The Universal Binary JSON specification has 3 goals:

1. **Universal Compatibility**

  Meaning absolute compatibility with the JSON spec itself as well as only
  utilizing data types that are natively supported in all popular programming
  languages.

  This allows 1:1 transforms between standard JSON and Universal Binary JSON as
  well as efficient representation in all popular programming languages without
  requiring parser developers to account for strange data types that their
  language may not support.

2. **Ease of Use**

  The Universal Binary JSON specification is intentionally defined using a
  single core data structure to build up the entire specification.

  This accomplishes two things: it allows the spec to be understood quickly and
  allows developers to write trivially simple code to take advantage of it or
  interchange data with another system utilizing it.

3. **Speed / Efficiency**

  Typically the motivation for using a binary specification over a text-based
  one is speed and/or efficiency, so strict attention was paid to selecting data
  constructs and representations that are (roughly) 30% smaller than their
  compacted JSON counterparts and optimized for fast parsing.

Got interested? Find more at [http://ubjson.org][ubjson]

[ubjson]: http://ubjson.org
[json]: http://json.org
[bson]: http://bsonspec.org
[bjson]: http://bjson.org
[smile]: http://wiki.fasterxml.com/SmileFormat
