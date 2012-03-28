******************************************************************************************************
Build reliable, traceable, distributed systems with ZeroMQ (ZeroRPC) by Jérôme Petazzoni from dotCloud
******************************************************************************************************

**Presenter**: `Jérôme Petazzoni
<https://us.pycon.org/2012/speaker/profile/261/>`_ (http://www.dotcloud.com/)
(`@jpetazzo <http://twitter.com/#!/jpetazzo>`_)

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/260/

**Slides**: https://docs.google.com/presentation/d/1FRh6kb79tcT0Deb4bjYVDvdvJAVMXYjXJK3EFT4pj8w/edit

**Video**: http://pyvideo.org/video/639/build-reliable-traceable-distributed-systems-wi

**Video running time**: 36:22

**ZeroRPC GitHub repo**: https://github.com/dotcloud/zerorpc-python

This talk is about `ZeroRPC <https://github.com/dotcloud/zerorpc-python>`_, a
library for doing RPC using `ZeroMQ <http://www.zeromq.org/>`_, that was
recently open-sourced by `dotCloud <http://dotcloud.com/>`_.

Introduction
============

Why did we build an RPC system?
-------------------------------

(00:22 / 36:22 into the video)

* `dotCloud`_ is a `PaaS <http://en.wikipedia.org/wiki/Platform_as_a_service>`_.

  - We deploy, monitor, and scale your apps (in the cloud!)

* Many moving parts

* ... On a large distributed cluster


What the architecture looks like
--------------------------------

(00:45 / 36:22 into the video)

.. image:: http://cl.ly/3k1X1O3I1i3r2P292h2f/Screen%20shot%202012-03-21%20at%2011.21.29%20PM.png


Easy Requirements
-----------------

(01:08 / 36:22 into the video)

* Expose arbitrary code with minimal modification

  - if we can do ``import foo; foo.bar(42)``
  - we want to be able to do ``foo = RemoteService(...); foo.bar(42)``

* Self-documented system

  - Want to see methods, signatures, and docstrings

More Difficult Requirements
---------------------------

(02:09 / 36:22 into the video)

* Propagate exceptions
* Language-agnostic
* Brokerless, highly available, fast, support fan-in/fan/out
  - Not necessarily all at the same time!
* We want to trace & profile nested calls

Why not {x}?
------------

(04:06 / 36:22 into the video)

* Why not HTTP
* Why not `AMQP
  <http://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol>`_

What we came up with
--------------------

(04:48 / 36:22 into the video)

`ZeroRPC <https://github.com/dotcloud/zerorpc-python>`_!

* Based on `ZeroMQ <http://www.zeromq.org/>`_ and `MessagePack
  <http://msgpack.org/>`_

* Supports everything we needed!

* Multiple implementations

  - Internal "reference" implementation in Python

  - Public "alternative" implementation with `gevent <http://www.gevent.org/>`_

    Internal `Node.js <http://nodejs.org/>`_ implementation (so-so)

Using it
========

Example: unmodified code
------------------------

(05:45 / 36:22 into the video)

* Expose the :mod:`urllib` module over RPC:

.. code-block:: bash

    $ zerorpc-client --server --bind tcp://127.0.0.1:1234 urllib

Example: calling code
---------------------

(06:14 / 36:22 into the video)

* From the command-line (for testing):

.. code-block:: bash

    $ zerorpc-client tcp://127.0.0.1:1234 quote "hello pycon"
    connecting to "tcp://127.0.0.1:1234"
    'hello%20pycon'

* From Python code:

.. code-block:: python

    >>> import zerorpc
    >>> remote_urllib = zerorpc.Client()
    >>> remote_urllib.connect('tcp://127.0.0.1:1234')
    [None]
    >>> remote_urllib.quote('hello pycon')
    'hello%20pycon'

Example: introspection
----------------------

(06:38 / 36:22 into the video)

We can list methods:

.. code-block:: bash

    $ zerorpc-client tcp://127.0.0.1:1234 | grep ^q
    quote                       quote('abc def') -> 'abc%20def'
    quote_plus                  Quote the query fragment of a URL; replacing ' ' with '+'

We can see signatures and docstrings:

.. code-block:: bash

    $ zerorpc-client tcp://127.0.0.1:1234 quote_plus -?
    connecting to "tcp://127.0.0.1:1234"

    quote_plus(s, safe='')

    Quote the query fragment of a URL; replacing ' ' with '+'

Example: exceptions
-------------------

(06:47 / 36:22 into the video)

.. code-block:: bash

    $ zerorpc-client tcp://127.0.0.1:1234 quote_plus
    connecting to "tcp://127.0.0.1:1234"
    Traceback (most recent call last):
    ...
    zerorpc.exceptions.RemoteError: Traceback (most recent call last):
      File "/Users/marca/dev/git-repos/zerorpc-python/zerorpc/core.py", line 201, in _async_task
        functor.pattern.process_call(self._context, socket, event, functor)
      File "/Users/marca/dev/git-repos/zerorpc-python/zerorpc/core.py", line 74, in process_call
        result = context.middleware_call_procedure(functor, *event.args)
      File "/Users/marca/dev/git-repos/zerorpc-python/zerorpc/context.py", line 88, in middleware_call_procedure
        return procedure(*args, **kwargs)
      File "/Users/marca/dev/git-repos/zerorpc-python/zerorpc/core.py", line 55, in __call__
        return self._functor(*args, **kargs)
    TypeError: quote_plus() takes at least 1 argument (0 given)

Example: load balancing
-----------------------

(07:07 / 36:22 into the video)

Start a load balancing hub:

.. code-block:: bash

    $ cat foo.yml
    in: "tcp://*:1111"
    out: "tcp://*:2222"
    type: queue
    $ zerohub.py foo.yml

Start (at least) one worker:

.. code-block:: bash

    $ zerorpc-client --server tcp://localhost:2222 urllib

Now connect to the "in" side of the hub:

.. code-block:: bash

    $ zerorpc-client tcp://localhost:1111

Example: high availability
--------------------------

(07:30 / 36:22 into the video)

Start a local `HAProxy <http://haproxy.1wt.eu/>`_ in TCP mode, dispatching
requests to 2 or more remote services or hubs:

.. code-block:: bash

    $ cat haproxy.cfg
    listen zerorpc 0.0.0.0:1111
        mode tcp
        server backend_a localhost:2222 check
        server backend_b localhost:3333 check
    $ haproxy -f haproxy.cfg

Start (at least) one backend:

.. code-block:: bash

    $ zerorpc-client --server --bind tcp://0:2222 urllib

Now connect to HAProxy:

.. code-block:: bash

    $ zerorpc-client tcp://localhost:1111

Non-example: PUB/SUB
--------------------

(08:01 / 36:22 into the video)

Not in public repo -- yet

* Broadcast a message to a group of nodes

  - But if a node leaves and rejoins, he'll lose messages

* Send a continuous stream of information

  - But if a speaker or listener leaves and rejoins...

You generally don't want to do this!

Better pattern: ZeroRPC streaming with `gevent`_

Example: streaming
------------------

(09:10 / 36:22 into the video)

* Server code returns an iterator
* Client code gets an iterator
* Small messages, high latency? No problem!
  - Server code will pre-push elements
  - Client code will notify server if pipeline runs low
* Huge messages? No problem!
  - Big data sets can be nicely chunked
  - They don't have to fit entirely in memory
  - Don't worry about timeouts anymore
* Also supports long polling

Example: tracing
----------------

(10:15 / 36:22 into the video)

Not in public repo yet

Implementation details
======================

(11:16 / 36:22 into the video)

This will be useful if:

* You think you might want to use ZeroRPC
* You think you might want to hack ZeroRPC
* You want to implement something similar
* You just happen to love distributed systems


ZeroMQ
------

(11:50 / 36:22 into the video)

* Sockets on steroids - http://zguide.zeromq.org/page:all
* Handles (re)connections for us
* Works over regular TCP
* Also has superfast ``ipc://`` and ``inproc://``
* Different patterns:

  - REQ/REP
  - PUB/SUB
  - PUSH/PULL
  - DEALER/ROUTER

* ``pip install pyzmq-static`` FTW (Thanks, `Brandon Craig Rhodes
  <http://rhodesmill.org/brandon/>`_!) (`pyzmq-static on PyPI
  <http://pypi.python.org/pypi/pyzmq-static/>`_)


MessagePack
-----------

(13:28 / 36:22 into the video)

* `MessagePack`_
* In our tests, msgpack is more efficient than JSON, BSON, YAML:

  - 20-50x faster
  - serialized output is 2x smaller or better

.. code-block:: bash

    $ pip install msgpack-python

.. code-block:: python

    >>> import msgpack
    >>> bytes = msgpack.dumps(data)


Wire format
-----------

(14:09 / 36:22 into the video)

Request: (headers, method_name, args)

* headers dict

  - no mandatory header
  - carries the protocol version number
  - used for tracing in our in-house version

* args

  - list of arguments
  - no named parameters

Response: (headers, ERR|OK|STREAM, value)


Timeouts
--------

(15:20 / 36:22 into the video)

* 0MQ does not detect disconnections
  (or rather, it works hard to hide them)
* You can't know when the remote is gone
* Original implementation: 30s timeout
* Published implementation: heartbeat


Introspection
-------------

(16:13 / 36:22 into the video)

* Expose a few special calls:

  - ``_zerorpc_list`` to list calls
  - ``_zerorpc_name`` to know who you're talking to
  - ``_zerorpc_ping`` (redundant with the previous one)
  - ``_zerorpc_help`` to retrieve the docstring of a call
  - ``_zerorpc_args`` to retrieve the argspec of a call
  - ``_zerorpc_inspect`` to retrieve everything at once


Naming
------

(17:10 / 36:22 into the video)

* Published implementation does not include any kind of naming/discovery

* In-house version uses a flat YAML file, mapping service names to 0MQ
  addresses and socket types, but ashamed to publish this :-)

* In progress: use DNS records

  - SRV for host+port
  - TXT for 0MQ socket type (not sure about this!)

* In progress: registration of services

  - `Majordomo protocol <http://rfc.zeromq.org/spec:7>`_

Security: there is none
-----------------------

(18:00 / 36:22 into the video)

* No security at all in 0MQ

  - assumes that you are on a private, internal network

* If you need to run "in the wild", use SSL:

  - bind 0MQ socket on localhost

  - run `stunnel <http://www.stunnel.org/>`_ (with client cert verification)

* In progress: authentication layer

* dotCloud API is actually ZeroRPC, exposed through a HTTP/ZeroRPC gateway

* In progress: standardization of this gateway

Tracing (not published yet)
---------------------------

(19:26 / 36:22 into the video)

* Initial implementation during a hack day

  - bonus: displays live latency and request rates, using http://projects.nuttnet.net/hummingbird/
  - bonus: displays graphical call flow, using http://raphaeljs.com/
  - bonus: send exceptions to `airbrake <https://bitbucket.org/greghball/django-airbrake>`_/`sentry <https://github.com/dcramer/sentry>`_

* Refactoring in progress, to "untie" it from the dotCloud infrastructure and Open Source It

How it works: all calls and responses are logged to a central place, along with
a trace_id unique to each sequence of calls.

Tracing: trace_id
-----------------

(21:14 / 36:22 into the video)

* Each call has a trace_id
* The trace_id is propagated to subcalls
* The trace_id is bound to a local context (think thread local storage)
* When making a call:

  - If there is a local trace_id, use it
  - If there is none ("root call"), generate one (GUID)

* trace_id is passed in all calls and responses

Note: this is not (yet) in the `GitHub repository
<https://github.com/dotcloud/zerorpc-python>`_:

Tracing: trace collection
-------------------------

(21:42 / 36:22 into the video)

* If a message (sent or received) has a trace_id, we send out the following things:

  - trace_id
  - call name (or, for return values, OK|ERR+exception)
  - current process name and hostname
  - timestamp

Internal details: the collection is built on top of the standard :mod:`logging`
module.

Tracing: trace storages
-----------------------

(22:10 / 36:22 into the video)

* Traces are sent to a `Redis <http://redis.io/>`_ key/value store

  - each trace_id is associated with a list of traces
  - we keep some per=service counters
  - Redis persistence is disabled
  - entries are given a TTL so they expire automatically
  - entries were initially JSON (for easy debugging)
  - ... then "compressed" with msgpack to save space
  - *approximately* 16 GB of traces per day

Internal details: the logging handler does not talk directly to Redis; it sends traces to a collector (which itself talks to Redis)

The problem with being synchronous
----------------------------------

(23:19 / 36:22 into the video)

* Original implementation was synchronous
* Long-running calls blocked the server
* Workaround: multiple workers and a hub
* Wastes resources
* Does not work well for *very long* calls

  - Deployment and provisioning of new cluster nodes
  - Deployment and scaling of user apps

Note: this is not specific to ZeroRPC (Preforking servers, threaded servers, WSGI...)

First shot at asynchronicity
----------------------------

(24:28 / 36:22 into the video)

* Send asynchronous events & setup callbacks
* "Please do ``foo(42)`` and send the result to this other place once you're done"
* We tried this. We failed.

  - distributed spaghetti code
  - trees falling in the forest with no one to hear them

* Might have worked better if we had...

  - better support in the library
  - better naming system
  - something to make sure we don't lose calls (a kind of distributed FSM, maybe?)

Gevent to the rescue!
---------------------

(26:02 / 36:22 into the video)

* Gevent -- http://www.gevent.org/
* Write synchronous code (a.k.a.: don't rewrite your services)
* Uses coroutines to achieve concurrency
* No forks, no threads (no problems? :-))
* Monkey patch standard library (to replace blocking calls with async versions)
* Achieve "unlimited" concurrency server-side

The version published on GitHub uses gevent.


Show me the code!
-----------------

(27:17 / 36:22 into the video)

https://github.com/dotcloud/zerorpc-python.git

.. code-block:: bash

    $ pip install git+git://github.com/dotcloud/zerorpc-python.git

Has:

* :mod:`zerorpc` module
* :mod:`zerorpc-client` helper
* exception propagation
* gevent integration

Doesn't have:

* tracing
* naming
* helpers for PUB/SUB and PUSH/PULL
* authentication


Questions?
----------

(28:25 / 36:22 into the video)

...







