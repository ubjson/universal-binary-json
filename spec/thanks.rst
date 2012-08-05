
Thanks
======

Below is a list of people that have submitted specific contributions,
corrections and implementations to help make the Universal Binary JSON
specification better.

Thank you all!

* `Alex Blewitt <http://twitter.com/#!/alblue>`_

  Helped catch a number of specification errors around UTF-8 encoding in the
  original draft of the specification that would have been confusing/nasty to
  release. He also provided great feedback about the size and performance
  metrics for the specification.

* `Alexander Shorin <http://code.google.com/p/simpleubjson/>`_

  Alex is both the author of the Python library and a valued collaborator on the
  Universal Binary JSON spec as it matured. Alex provided instrumental insight
  into the modifications made between Draft 8 and Draft 9 of the spec to help
  simplify the spec by removing all the duplicate (compact) type
  representations, simplifying the length-arguments for `STRING` and `HUGE` as
  well as being the one to point out that the length arguments for the `ARRAY`
  and `OBJECT` container types are effectively useless once the streaming-format
  support was added (and do not make generator code or parsing code any easier
  or more performant).

* `John Cowan <http://tech.groups.yahoo.com/group/json/message/1734>`_

  John was the one that recommended using UTF-8 string-encoded values
  (or `huge`) for arbitrarily huge numbers after seeing my desire to avoid
  including any non-portable constructs into the binary format.

  Given that the discussion on numeric formats had been a very active one with
  lots of feelings on all sides, it was a boon to have John step up with such a
  simple suggestion that allowed for maximum compatibility and portability.
  It was a win-win all the way around.

* `Michael Makarenko <http://www.m1xa.com/>`_ (aka “M1xA”)

  Michael is the author behind the `Ubjson.NET <http://ubjsonnet.codeplex.com/>`_
  library and contributor of the `int16` and `float` numeric types to the
  specification. For numeric-heavy (e.g. scientific) data, the inclusions of the
  `int16` and `float` types can lead to significant space savings when writing
  out values in the Universal Binary JSON format.

  Michael has also gone to great lengths to make the .NET implementation of
  UBJSON as tight and performant as possible; collaborating on benchmark design
  and testing data as well as compatibility testing between implementations to
  ensure a great Universal Binary JSON experience for .NET developers.

  In addition to development, Michael has helped contribute to the growth of the
  Universal Binary JSON community with
  `articles about the specification <http://habrahabr.ru/blogs/open_source/130112/>`_.

* `Paul Davis <http://davispj.com/>`_

  While approaching the CouchDB team for feedback on the Universal Binary JSON
  spec, I met Paul who was willing to spend a significant amount of time
  reviewing the specification and recommending suggestions, changes and
  improvements from everything the CouchDB team has learned by dealing closely
  with JSON for years.

  Paul was the brains behind the compacted type presentation
  (``s``, ``h``, ``a`` and ``o``) using a single byte instead of 3 bytes to
  represent the length of an entity which was something the spec had originally
  avoided due to complexity, but as Paul clarified at-scale (e.g. a huge CouchDB
  data store) those few bytes in some data sets that are working with very small
  values (like string keywords) can really add up.

  Paul also pointed out the shortcomings of prefixing the length to the two
  container types if the specification could ever be used easily with services
  or apps that streamed UBJ format for huge runs of data that the server
  couldn't load, buffer and count ahead of time before responding to the client.
  In order to more easily support streaming, unknown-length container types had
  to be added.

  Paul also pointed out the importance of a ``NO_OP``/``SKIP``/``IGNORE`` type
  that can be useful during a long-lived streaming operation where the server
  may be waiting on something (like a DB) and you need to keep the connection
  alive between client/server and avoid the client timing out, but you need the
  client to know the data it is receiving is just meant as a “Hang on” message
  from the server and not actual data. This is where the ``NO_OP`` command comes
  in handy.

* `Stephan Beal <http://tech.groups.yahoo.com/group/json/message/1686>`_

  Stephan helped quite a bit with understanding the implications of a >= 64-bit
  numeric format and the implications of portability across a number of popular
  platforms.

* `JSON Specification Group <http://json.org>`_

  I would like to personally thank everyone in the JSON Specification Group.
  The amount of feedback and help with the specification has been wonderful,
  constructive and creative. It also lead to one of the busiest conversations
  in the last year!
