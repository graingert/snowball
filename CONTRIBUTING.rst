Adding a new stemming algorithm
===============================

This needs PRs against three repositories.  Name the branch the same for
at least `snowball` and `snowball-data`, push to the latter repo first, and the
CI should use your new vocabulary list when running the testsuite.

Some points to note about algorithm implementations:

* Avoid literal non-ASCII characters in snowball string literals - they will
  work OK for languages that use UTF-8, but not wide-character Unicode or other
  encodings.  Instead use ``stringdef`` like the existing stemmers do, and
  please use the newer `U+` notation rather than the older ``hex`` or
  ``decimal`` as this allows us to support different encodings without having
  to modify the source files - for example::

    stringdef o" {U+00F6}
    define foo 'o{o"}'

  not::

    stringdef o" hex F6
    define foo 'o{o"}'

  and definitely not::

    define foo 'oö'

  It's helpful to consistently use the same ``stringdef`` codes across the
  different stemmers - the website has `guidance on what to use
  <https://snowballstem.org/codesets/guide.html>`_ and a `list of stringdef
  lines for common characters to cut and paste from
  <https://snowballstem.org/codesets/latin-stringdef-list.txt>`_.

snowball repo
-------------

Add `.sbl` source to algorithms subdirectory.

Add entry to `libstemmer/modules.txt`, maintaining the current sorted order by
the first column.  The columns are:

* Algorithm name (needs to match the `.sbl` source without extension)
* Encodings to support.  Wide-character Unicode is always supported
  and doesn't need to be listed here.  You should always include `UTF_8`, and
  also `ISO_8859_1` if the stemmer only uses characters from that and the
  language can be usefully written using it.  We currently also have support
  for `ISO_8859_2` and `KOI8_R`, but other single-byte character sets can be
  supported quite easily if they are useful.
* Names and ISO-639 codes for the language.  Wikipedia has a handy list of `all
  the ISO-639 codes <https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes>`_ -
  find the row for your new language and include the codes from the "639-1",
  "639-2/T" and (if different) "639-2/B" columns.  For example, for the `Afar`
  language you'd put `afar,aa,aar` here.

snowball-data repo
------------------

Add subdirectory named after new stemmer containing:

* voc.txt - word list
* output.txt - stemmed equivalents
* COPYING - licensing details (word lists need to be under an OSI-approved
  licence)

If you don't have access to a suitably licensed word list of a suitable size,
you may be able to use the `wikipedia-most-common-words` script to generate
one by extracting the most frequent words from a Wikipedia dump in the
language the stemmer is for.  If the language uses a script/alphabet which
isn't already supported you may need to add a regular new regular expression.

snowball-website repo
---------------------

Create subdirectory of `algorithms/` named after the language.

Create `stemmer.tt` which describes the stemming algorithm.  This is a
"template toolkit" template which is essentially a mix of HTML and some
macros for adding the navigation, sample vocabulary, etc.  See the
existing `stemmer.tt` files for other algorithms for inspiration.

If it is based on an academic paper, cite the paper and describe any difference
between your implementation and that described in the paper (for example,
sometimes papers have ambiguities that need resolving to re-implement the
algorithm described).

If you have a stopword list, add that as `stop.txt` and link to it from
`stemmer.tt`.

Link to your new `stemmer.tt` from `algorithms/index.tt`.

Add a news entry to `index.tt`.

.. FIXME: Also needs adding for the online demo.

Adding a new programming language backend
=========================================

Copy an existing `compiler/generator_*.c` for your new language and modify
away (`generator.c` has the generator for C, but also some common functions
so if you start from this one you'll need to remove those common functions).
Please resist reformatting existing code - there's currently a lot of code
repeated in each generator which ought to be pulled out as common code, and
if you reformat that just makes that job harder.

Add your new source to `COMPILER_SOURCES` in `GNUmakefile`.

Add prototypes for the new functions to `compiler/header.h`.

Add support to `compiler/driver.c`.

Add targets to `GNUmakefile` to run tests for the new language.

Hook up automated testing via CI in `.travis.yml`.
