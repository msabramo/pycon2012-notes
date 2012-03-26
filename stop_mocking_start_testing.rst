***************************************************************************
Stop Mocking, Start Testing
***************************************************************************

**Presenters**: `Augie Fackler
<https://us.pycon.org/2012/speaker/profile/219/>`_ and `Nathaniel Manista
<https://us.pycon.org/2012/speaker/profile/295/>`_ of `Google Code
<http://code.google.com/>`_

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/315/

**Slides**: https://code.google.com/a/google.com/p/stop-mocking-start-testing/

**Video**: http://pyvideo.org/video/629/stop-mocking-start-testing

**Video running time**: 34:53


**Lessons**

`(04:18) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=4m18s>`_

* Users are not a test infrastructure.
* Coded tests with greater human costs than CPU costs are also not a test infrastructure.
* A project simply cannot grow this way.

`(07:59) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=7m59s>`_

.. code-block:: python

    class Real(object):
        def EnLolCat(self, phrases, words=None, kitteh=None):
            # ...

    class Fake1(object):
        def EnLolCat(self, phrases, words):
            # ...

    class Fake2(object):
        def EnLolCat(self, phrases):
            # ...

`(08:45) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=8m45s>`_

"Mock objects tell you what you want to hear."

Tests run only against mock objects

Python doesn't check that a mock is true to what it is mocking

Developers don't either!


`(09:48) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=9m48s>`_

**Lessons**

* Share mocks among test modules.
* When choosing between zero and one mock, try zero!

`(10:33) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=10m33s>`_

**If you need a mock, have exactly one well-tested mock.**

`(12:23) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=12m23s>`_

**Use full system tests to make up for gaps in unit- level coverage**

`(13:13) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=13m13s>`_

**Full system tests are not a replacement for unit-level tests!**

`(13:29) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=13m29s>`_

**Test stories with full system tests**


Modern Mocking
--------------

`(14:29) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=14m29s>`_

A collection of...

... authoritative...

... narrow...

... isolated...

... fakes.

You don't want mocks spread out all over the place that need to be
updated whenever the real object changes.

Don't mock things that don't need to be mocked - things that are cheap like
simple data structures or things that are stateless or simple.

They use what they call "fakes" and don't find distinctions between doubles,
stubs, mocks, etc. useful.

They don't really use declarative/fluent mocks that check that they're being
called the right # of times, etc. It sounds like they have formal mock classes.


Testing Today
-------------

`(17:58) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=17m58s>`_

Tests are written to the interface, not the implementation

Unit tests are run against both mock and real implementations

System tests are run in continuous integration and quality assurance


Design For Love *And* Testing
=============================

`(20:30) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=20m30s>`_

* Inject object dependencies as required construction parameters.
* Separate state from behavior
* Define interfaces between components.
* Decline to write a test when there's no clear interface.


Injected dependencies
---------------------

`(20:38) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=20m38s>`_

.. code-block:: python

    # Not this:
    class BadView(BaseView):
        def __init__(self, database=None):
            if database is None:
                # This reads a global value set by
                # a command line flag
                database = DefaultDatabase()
            self._database = database

    # This:
    class GoodView(BaseView):
        def __init__(self, database):
            self._database = database




They used to have *optional* injected dependencies -- i.e. if you didn't
provide an argument with the dependency it would choose one automatically
(either hard-coded or use something from a command-line option, config file,
etc.)


Separate state from behavior
----------------------------

`(21:34) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=21m34s>`_

Separate state (especially storage) from behavior -- i.e.: if a method has a
part that touches object attributes and a part that doesn't; factor out the
parts that don't touch the attributes into a separate "free function".


Define interfaces between components
------------------------------------

`(23:29) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=23m29s>`_


Decline to write a test when there's no clear interface
-------------------------------------------------------

`(23:54) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=23m54s>`_


Thank you
---------

`(24:50) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=24m50s>`_


Questions
---------

`(25:21) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=25m21s>`_

Someone asked about adding tests to legacy code and drawing a line in the sand.

`(28:41) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=28m41s>`_

I mentioned `"Working Effectively with Legacy Code" by Michael Feathers
<http://amzn.to/AyKH75>`_ for a guy who asked about adding tests to untested
code.

I asked the speakers about `"Tell, Don't Ask"
<http://pragprog.com/articles/tell-dont-ask>`_ and they were not familiar with
it, so I don't think it's something that they adhere strongly to.

`(30:04) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=30m04s>`_

An interesting point someone made is that it is nice to be able to check that
mocks adhere to interfaces, e.g..: using `ABCs
<http://docs.python.org/library/abc.html>`_ or `zope.interface
<http://docs.zope.org/zope.interface/>`_. This could probably be generalized to
languages like PHP. For example, this might be an
argument in favor of using formal interfaces over duck typing.

`(31:40) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=31m40s>`_

Q: How to decide what to unit test and what to system test?

`(33:49) <https://www.youtube.com/watch?v=Xu5EhKVZdV8#t=33m49s>`_

Mocks vs. fakes - they treat mocks as a last resort - they don't write mocks
for their own classes.

