.. _user:

Quickstart Guide
================

First, congratulations on picking ``hyper`` for your HTTP/2.0 needs. ``hyper``
is the premier (and, as far as we're aware, the only) Python HTTP/2.0 library.
In this section, we'll walk you through using ``hyper``.

Installing hyper
----------------

To begin, you will need to install ``hyper``. This can be done like so:

.. code-block:: bash

    $ pip install hyper

If ``pip`` is not available to you, you can try:

.. code-block:: bash

    $ easy_install hyper

If that fails, download the library from its GitHub page and install it using:

.. code-block:: bash

    $ python setup.py install

Installation Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~

There are no specific requirements for using `hyper`, beyond the fact that it
supports Python versions 2.7 and 3.3+ on CPython and PyPy.

Making Your First Request
-------------------------

With ``hyper`` installed, you can start making HTTP/2.0 requests. At this
stage, ``hyper`` can only be used with services that *definitely* support
HTTP/2.0. Before you begin, ensure that whichever service you're contacting
definitely supports HTTP/2.0. For the rest of these examples, we'll use
Twitter.

Begin by getting the Twitter homepage::

    >>> from hyper import HTTP20Connection
    >>> c = HTTP20Connection('twitter.com:443')
    >>> c.request('GET', '/')
    1
    >>> resp = c.getresponse()

Used in this way, ``hyper`` behaves exactly like ``http.client``. You can make
sequential requests using the exact same API you're accustomed to. The only
difference is that
:meth:`HTTP20Connection.request() <hyper.HTTP20Connection.request>` returns a
value, unlike the equivalent ``http.client`` function. The return value is the
HTTP/2.0 *stream identifier*. If you're planning to use ``hyper`` in this very
simple way, you can choose to ignore it, but it's potentially useful. We'll
come back to it.

Once you've got the data, things continue to behave exactly like
``http.client``::

    >>> resp.getheader('content-encoding')
    'deflate'
    >>> resp.getheaders()
    [('x-xss-protection', '1; mode=block')...
    >>> resp.status
    200

We know that Twitter has compressed the response body. ``hyper`` will
automatically decompress that body for you, no input required::

    >>> body = resp.read()
    b'<!DOCTYPE html>\n<!--[if IE 8]><html clas ....

That's all it takes.

Streams
-------

In HTTP/2.0, connections are divided into multiple streams. Each stream carries
a single request-response pair. You may start multiple requests before reading
the response from any of them, and switch between them using their stream IDs.

For example::

    >>> from hyper import HTTP20Connection
    >>> c = HTTP20Connection('twitter.com:443')
    >>> first = c.request('GET', '/')
    >>> second = c.request('GET', '/lukasaoz')
    >>> third = c.request('GET', '/about')
    >>> second_response = c.getresponse(second)
    >>> first_response = c.getresponse(first)
    >>> third_response = c.getresponse(third)

``hyper`` will ensure that each response is matched to the correct request.

Requests Integration
--------------------

Do you like `requests`_? Of course you do, everyone does! It's a shame that
requests doesn't support HTTP/2.0 though. To rectify that oversight, ``hyper``
provides a transport adapter that can be plugged directly into Requests, giving
it instant HTTP/2.0 support.

All you have to do is identify a host that you'd like to communicate with over
HTTP/2.0. Once you've worked that out, you can get started straight away::

    >>> import requests
    >>> from hyper.contrib import HTTP20Adapter
    >>> s = requests.Session()
    >>> s.mount('https://twitter.com', HTTP20Adapter())
    >>> r = s.get('https://twitter.com')
    >>> print(r.status_code)
    200

This transport adapter is subject to all of the limitations that apply to
``hyper``, and provides all of the goodness of requests.

A quick warning: some hosts will redirect to new hostnames, which may redirect
you away from HTTP/2.0. Make sure you install the adapter for all the hostnames
you're interested in::

    >>> a = HTTP20Adapter()
    >>> s.mount('https://twitter.com', a)
    >>> s.mount('https://www.twitter.com', a)

.. _requests: http://python-requests.org/
