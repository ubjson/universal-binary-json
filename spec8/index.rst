.. Universal Binary JSON documentation master file, created by
   sphinx-quickstart on Sat Aug  4 16:26:00 2012.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Universal Binary JSON
=====================

`JSON`_ has become a ubiquitous text-based file format for data interchange.
Its simplicity, ease of processing and (relatively) rich data typing made it a
natural choice for many developers needing to store or shuffle data between
systems quickly and easy.

Unfortunately, marshaling native programming language constructs in and out of
a text-based representations does have a measurable processing cost associated
with it.

In high-performance applications, avoiding the text-processing step of JSON can
net big wins in both processing time and size reduction of stored information,
which is where a binary JSON format becomes helpful.

.. toctree::
  :maxdepth: 3

  spec.rst
  type_reference.rst
  libraries.rst
  thanks.rst

Why UBJSON?
-----------

Attempts to make using JSON faster through binary specifications like
`BSON`_, `BJSON`_ or `Smile`_ exist, but have been `rejected`_
from `mass-adoption`_ for two reasons:

* Custom (Binary-Only) Data Types:
  Inclusion of custom data types that have no ancillary in the original JSON
  spec, leaving room for incompatibilities to exist as different implementations
  of the spec handle the binary-only data types differently.

* Complexity: Some specifications provide higher performance or smaller
  representations at the cost of a `much more complex specification`_,
  making implementations more difficult which can slow or block adoption. One of
  the key reasons JSON became as popular as it did was because of its ease of
  use.

BSON, for example, defines types for binary data, regular expressions,
JavaScript code blocks and other constructs that have no equivalent data type in
JSON. BJSON defines a binary data type as well, again leaving the door wide open
to interpretation that can potentially lead to incompatibilities between two
implementations of the spec and Smile, while the closest, defines more complex
data constructs and generation/parsing rules in the name of absolute space
efficiency.

The existing binary JSON specifications all define incompatibilities or
complexities that undo the singular tenant that made JSON so successful:
**simplicity**.

JSON’s simplicity made it accessible to anyone, made implementations in every
language available and made explaining it to anyone consuming your data
immediate.

Any successful binary JSON specification must carry these properties forward for
it to be genuinely helpful to the community at large.

This specification is defined around a singular construct used to build up and
represent JSON values and objects. Reading and writing the format is trivial,
designed with the goal of being understood in under 10 minutes (likely less if
you are very comfortable with JSON already).

Fortunately, while the Universal Binary JSON specification carriers these
tenants of simplicity forward, it is also able to take advantage of optimized
binary data structures that are (on average) 30% smaller than compacted JSON and
specified for ultimate read performance; bringing **simplicity**, **size** and
**performance** all together into a single specification that is 100% compatible
with JSON.

Why not JSON+gzip?
------------------

On the surface simply gzipping your compacted JSON may seem like a valid (and
smaller) alternative to using the Universal Binary JSON specification, but there
are two significant costs associated with this approach that you should be aware
of:

#. At least a `50% performance overhead`_ for processing the data.
#. Lack of data clarity and inability to inspect it directly.

While gzipping your JSON will give you great compression, about 75% on average,
the overhead required to read/write the data becomes significantly higher.
Additionally, because the binary data is now in a compressed format you can no
longer open it directly in an editor and scan the human-readable portions of it
easily; which can be important during debugging, testing or data verification
and recovery.

Utilizing the Universal Binary JSON format will typically provide a
30% reduction in size and store your data in a read-optimized format offering
you much higher performance than even compacted JSON. If you had a usage
scenario where your data is put into long-term cold storage and pulled out in
large chunks for processing, you might even consider gzipping your
Universal Binary JSON files, storing those, and when they are pulled out and
unzipped, you can then process them with all the speed advantages of UBJ.

As always, deciding which approach is right for your project depends heavily on
what you need.

Goals
-----

The `Universal Binary JSON`_ specification has 3 goals:

#. **Universal Compatibility**

   Meaning absolute compatibility with the JSON spec itself as well as only
   utilizing data types that are natively supported in all popular programming
   languages.

   This allows 1:1 transforms between standard JSON and Universal Binary JSON as
   well as efficient representation in all popular programming languages without
   requiring parser developers to account for strange data types that their
   language may not support.

#. **Ease of Use**

   The Universal Binary JSON specification is intentionally defined using a
   single core data structure to build up the entire specification.

   This accomplishes two things: it allows the spec to be understood quickly and
   allows developers to write trivially simple code to take advantage of it or
   interchange data with another system utilizing it.

#. **Speed / Efficiency**

   Typically the motivation for using a binary specification over a text-based
   one is speed and/or efficiency, so strict attention was paid to selecting
   data constructs and representations that are (roughly) 30% smaller than their
   compacted JSON counterparts and optimized for fast parsing.

Indices and tables
==================

* :ref:`genindex`
* :ref:`search`

.. _JSON: http://json.org
.. _UBJSON: http://ubjson.org
.. _Universal Binary JSON: http://ubjson.org
.. _BSON: http://bsonspec.org
.. _BJSON: http://bjson.org
.. _Smile: http://wiki.fasterxml.com/SmileFormat
.. _rejected: https://issues.apache.org/jira/browse/COUCHDB-702
.. _mass-adoption: http://bsonspec.org/#/implementation
.. _much more complex specification: http://wiki.fasterxml.com/SmileFormatSpec
.. _50% performance overhead: http://www.cowtowncoder.com/blog/archives/2009/05/entry_263.html
