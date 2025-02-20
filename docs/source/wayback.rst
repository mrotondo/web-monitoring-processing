.. currentmodule:: web_monitoring.internetarchive

**********************************************
Python API to Internet Archive Wayback Machine
**********************************************

Search for historical mementos (archived copies) of a URL. Download metadata
about the mementos and/or the memento content itself.

Tutorial
========

What is the earliest memento of nasa.gov?
-----------------------------------------

Instantiate a :class:`WaybackClient`.

.. ipython:: python

   from web_monitoring.internetarchive import WaybackClient
   client = WaybackClient()

Search for all Wayback's records for nasa.gov.

.. ipython:: python

   results = client.search('nasa.gov')

This statement should execute fairly quickly because it doesn't actually do
much work. The object we get back, ``results``, is a *generator*, a "lazy"
object from which we can pull results, one at a time. As we pull items
out of it, it loads them as needed from the Wayback Machine in chronological
order. We can see that ``results`` by itself is not informative:

.. ipython:: python

   results

There are couple ways to pull items out of generator like ``results``. One
simple way is to use the built-in Python function :func:`next`, like so:

.. ipython:: python

   record = next(results)

This takes a moment to run because, now that we've asked to see the first item
in the generator, this lazy object goes to fetch a chunk of results from the
Wayback Machine. Looking at the record in detail,

.. ipython:: python

   record

we can find our answer: Wayback's first memento of nasa.gov was in 1996. We
can use dot access on ``record`` to access the date specifically.

.. ipython:: python

   record.date

How many times does the word 'mars' appear on nasa.gov?
-------------------------------------------------------

Above, we access the metadata for the oldest memento on nasa.gov, stored in
the variable ``record``. Starting from where we left off, we'll access the
*content* of the memento and do a very simple analysis.

The Wayback Machine provides two ways to look at the data it has captured.
There is a copy edited for human viewers on the web, available at the record's
``view_url``, and there is the original copy of what was captured when the page
was originally scraped, availabe at the record's ``raw_url``. For analysis
purposes, we generally want the ``raw_url``.

Let's download the raw content using ``WaybackClient``. (You could download the
content directly with an HTTP library like ``requests``, but ``WaybackClient``
adds extra tools for dealing with Wayback Machine servers.)

.. ipython:: python

   response = client.get_memento(record.raw_url)
   content = response.content.decode()

We can use the built-in method ``count`` on strings to count the number of
times that ``'mars'`` appears in the content.

.. ipython:: python

   content.count('mars')

This is case-sensitive, so to be more accurate we should convert the content to
lowercase first.

.. ipython:: python

   content.lower().count('mars')

We picked up a couple additional occurrences that the original count missed.

We have been doing this count on the full HTML source of the page, which
includes both visible text and code that is not visible to a human visitor
browsing the page. We can use a function in ``web_monitoring`` to extract only
the visible text.

.. ipython:: python

   from web_monitoring.differs import _get_visible_text
   visible = _get_visible_text(content)

To get a quick sense of the difference between ``content`` and ``visible``
let's look at the first 80 characters of each.

.. ipython:: python

   print(content[:80])
   print(visible[:80])

And now let's do the word count on just the visible content.

.. ipython:: python

   visible.lower().count('mars')

Our count of ``visible`` excludes appearances of ``'mars'`` that aren't in the
visible text, such as in link URLs. Both analyses could be interesting,
depending on the context.

API Documentation
=================

The Wayback Machine exposes its data through two different mechanisms,
implementing two different standards for archival data, the CDX API and the
Memento API. We implement a Python client that can speak both.

.. autoclass:: WaybackClient

    .. automethod:: search
    .. automethod:: list_versions
    .. automethod:: get_memento
    .. automethod:: timestamped_uri_to_version

.. autoclass:: WaybackSession

    .. automethod:: reset
