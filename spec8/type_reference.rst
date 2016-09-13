
Type reference
++++++++++++++

The table below is a quick-reference for folks working closely with the
Universal Binary JSON format that want all the information at their finger tips:

+-----------------+--------------------------+--------+---------+--------------+
|       Type      |           Size           | Marker | Length? |     Data?    |
+=================+==========================+========+=========+==============+
| null            | 1-byte                   | Z      | No      | No           |
+-----------------+--------------------------+--------+---------+--------------+
| true            | 1-byte                   | T      | No      | No           |
+-----------------+--------------------------+--------+---------+--------------+
| false           | 1-byte                   | F      | No      | No           |
+-----------------+--------------------------+--------+---------+--------------+
| byte            | 2-bytes                  | B      | No      | Yes          |
+-----------------+--------------------------+--------+---------+--------------+
| int16           | 3-bytes                  | i      | No      | Yes          |
+-----------------+--------------------------+--------+---------+--------------+
| int32           | 5-bytes                  | I      | No      | Yes          |
+-----------------+--------------------------+--------+---------+--------------+
| int64           | 9-bytes                  | L      | No      | Yes          |
+-----------------+--------------------------+--------+---------+--------------+
| float (32-bit)  | 5-bytes                  | d      | No      | Yes          |
+-----------------+--------------------------+--------+---------+--------------+
| double (64-bit) | 9-bytes                  | D      | No      | Yes          |
+-----------------+--------------------------+--------+---------+--------------+
| huge (number)   | 2-bytes                  | h      | Yes     | Yes          |
|                 | + byte length of string  |        |         | if non-empty |
+-----------------+--------------------------+--------+---------+--------------+
| huge (number)   | 5-bytes                  | H      | Yes     | Yes,         |
|                 | + byte length of string  |        |         | if non-empty |
+-----------------+--------------------------+--------+---------+--------------+
| string          | 2-bytes                  | s      | Yes     | Yes,         |
|                 | + byte length of string  |        |         | if non-empty |
+-----------------+--------------------------+--------+---------+--------------+
| string          | 5-bytes                  | S      | Yes     | Yes,         |
|                 | + byte length of string  |        |         | if non-empty |
+-----------------+--------------------------+--------+---------+--------------+
| array           | 2-bytes                  | a      | Yes     | Yes,         |
|                 | + byte length of string  |        |         | if non-empty |
+-----------------+--------------------------+--------+---------+--------------+
| array           | 5-bytes                  | A      | Yes     | Yes,         |
|                 | + byte length of string  |        |         | if non-empty |
+-----------------+--------------------------+--------+---------+--------------+
| object          | 2-bytes                  | o      | Yes     | Yes          |
|                 | + byte length of string  |        |         | if non-empty |
+-----------------+--------------------------+--------+---------+--------------+
| object          | 5-bytes                  | O      | Yes     | Yes,         |
|                 | + byte length of string  |        |         | if non-empty |
+-----------------+--------------------------+--------+---------+--------------+
| noop            | 1-byte                   | N      | No      | No           |
+-----------------+--------------------------+--------+---------+--------------+
| end             | 1-byte                   | E      | No      | No           |
+-----------------+--------------------------+--------+---------+--------------+

Numeric Types
=============

All numeric types are signed.

floats (32-bit)
---------------

All 32-bit float values are written into the binary format using the
`IEEE 754 single precision floating point format`_, which is the following
structure:

* Bit 31 (1 bit) – sign
* Bit 30-23 (8 bits) – exponent
* Bit 22-0 (23 bits) – fraction (significand)

doubles (64-bit)
----------------

All 64-bit double values are written into the binary format using the
`IEEE 754 double precision floating point format`_, which is the following
structure:

* Bit 63 (1 bit) – sign
* Bit 62-52 (11 bits) – exponent
* Bit 51-0 (52 bits) – fraction (significand)

String Encoding
===============

All `string` values (which includes `huge` values since they are string-encoded)
must be `UTF-8`_ encoded.

This provides a `number of advantages`_ and inter-compatibility across systems and
alternative data formats.

Arrays & Objects
================

The `length` argument specified is the `number of child elements` the parent
container contains. A `child element` is defined as:

* in an `object`, a single name-value pair.
* in an `array`, a single value.

For example:

* if an array contains 4 integers, the `length` of that array is 4.
* if an object contains 4 name-value pairs, the `length` of that object is 4.
* if an array contains 13 `User objects`, the `length` of the array is 13.
* if an object contains 7 arrays, the `length` of the object is 7.

.. note::

  Universal Binary JSON is a :ref:`streaming-friendly <streaming>` specification
  and supports the use of :ref:`unknown-length container <unsized_container>`
  types if you need them!

Support for ‘huge’ Numeric Type
===============================

The huge data type is an ultra-portable mechanism by which arbitrarily long
numbers ``> 64-bit`` in size (integer or decimal) can be passed between systems
that support them and degraded gracefully in systems that do not support them.

.. note::

  `huge` values are **only** meant to store values ``> 64-bit`` in size.
  It is in violation of the Universal Binary JSON specification to store a value
  ``<= 64-bits`` as a huge.

  This design was chosen intentionally as it greatly simplifies (and optimizes)
  the generation and parsing code for the UBJ format as no introspection of the
  `huge` value is necessary for a platform to try and marshal them into a
  smaller format.

  This way the parsing code becomes simple, either creating an arbitrarily large
  number out of the value (e.g. `BigDecimal`_ in Java), returns an error to the
  caller because of an unsupported type or optionally skips the unsupported data
  and continues parsing.

`huge` values must be written out in accordance with the original
`JSON number specification`_.

Many programming languages have native support for arbitrarily large numbers,
but many do not. If you are working in an environment that does not support
numbers > 64-bit numbers, please see our recommendation on handling them in the
:ref:`Best Practices <best_practices>` section.

Optimized Storage Size
======================

All variable-length value types (`string`, `huge`, `array`, `object`) have a
more compact representation using 1-byte (instead of 4-bytes) for the `length`
argument when the `length` value is <= 254.

These more compact types always use the lowercased version of the `marker`
ASCII char. For example, ``a`` for `array`, ``s`` for `string` and so on.

.. warning::

  When using the compact representations of these different types, remember that
  the `length` must be ``<= 254`` because the `length` of 255 (``0xFF``) has a
  special meaning when it comes to `array` and `object` types.

noop and Streaming Support
==========================

The :ref:`noop type <noop>` is a general purpose type that has no meaning, but
is mostly commonly used in streaming scenarios where a server must send a client
a `keep alive` message.

To support this use-case, the specification needed to support a special type
that meant nothing, so a server and client could make use of it without
polluting the actual data that was being exchanged.

.. warning::

  The `noop` type can be used for other purposes or signals as well, but it is
  defined to have no value and no effect on the data it may be included in.

  The `noop` type is meant to be sent between discrete values in a streaming
  scenario and can never be sent inside of the byte-data that makes up a single
  value.

  For example, if a server is writing a string “Hello World” back to the client,
  the server must write the entire ``[s][11][Hello World]`` sequence back to the
  client unbroken; a `noop` marker cannot be sent inside of that value.

  `noop` markers must only be written between values being transmitted (e.g.
  between values in an `array` or between the name and value pair inside of an
  `object`).

Examples
========

Please see the :ref:`value_types` and :ref:`container_types` sections of the
specification for examples.

.. _IEEE 754 single precision floating point format: http://en.wikipedia.org/wiki/IEEE_754-1985
.. _IEEE 754 double precision floating point format: http://en.wikipedia.org/wiki/Double_precision_floating-point_format#Double_precision_binary_floating-point_format
.. _UTF-8: http://en.wikipedia.org/wiki/UTF-8
.. _number of advantages: http://en.wikipedia.org/wiki/UTF-8#Advantages
.. _BigDecimal: http://download.oracle.com/javase/6/docs/api/java/math/BigDecimal.html
.. _JSON number specification: http://json.org
