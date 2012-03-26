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


**Lessons**

* Users are not a test infrastructure.
* Coded tests with greater human costs than CPU costs are also not a test infrastructure.
* A project simply cannot grow this way.

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

"Mock objects tell you what you want to hear."

Tests run only against mock objects

Python doesn't check that a mock is true to what it is mocking

Developers don't either!


**Lessons**

* Share mocks among test modules.
* When choosing between zero and one mock, try zero!

**If you need a mock, have exactly one well-tested mock.**

**Use full system tests to make up for gaps in unit- level coverage**

**Full system tests are not a replacement for unit-level tests!**


Modern Mocking
--------------

A collection of...

... authoritative...

... narrow...

... isolated...

... fakes.


Testing Today
-------------

Tests are written to the interface, not the implementation

Unit tests are run against both mock and real implementations

System tests are run in continuous integration and quality assurance


Design For Love And Testing
===========================

* Inject object dependencies as required construction parameters.
* Separate state from behavior
* Define interfaces between components.
* Decline to write a test when there's no clear interface.


Injected dependencies
---------------------

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




Mocks should be:

* shared
* collected in one place
* authoritative

because you don't want mocks spread out all over the place that need to be
updated whenever the real object changes.

Don't mock things that don't need to be mocked - things that are cheap like
simple data structures or things that are stateless or simple.

They use what they call "fakes" and don't find distinctions between doubles,
stubs, mocks, etc. useful.

They don't really use declarative/fluent mocks that check that they're being
called the right # of times, etc. It sounds like they have formal mock classes.

They used to have *optional* injected dependencies -- i.e. if you didn't
provide an argument with the dependency it would choose one automatically
(either hard-coded or use something from a command-line option, config file,
etc.)

Separate state (storage) from behavior -- i.e.: if a method has a part that
touches object attributes and a part that doesn't; factor out the parts that
don't touch the attributes into a separate "free function".

I mentioned `"Working Effectively with Legacy Code" by Michael Feathers
<http://amzn.to/AyKH75>`_ for a guy who asked about adding tests to untested
code.

I asked the speakers about `"Tell, Don't Ask"
<http://pragprog.com/articles/tell-dont-ask>`_ and they were not familiar with
it, so I don't think it's something that they adhere strongly to.

An interesting point someone made is that it is nice to be able to check that
mocks adhere to interfaces, e.g..: using `ABCs
<http://docs.python.org/library/abc.html>`_ or `zope.interface
<http://docs.zope.org/zope.interface/>`_. This could probably be generalized to
languages like PHP. For example, this might be an
argument in favor of using formal interfaces over duck typing.

Mocks vs. fakes - they treat mocks as a last resort - they don't write mocks
for their own classes.

