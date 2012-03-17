***************************************************************************
Speedily Practical Large-Scale Tests
***************************************************************************

**Presenter**: `Erik Rose
<https://us.pycon.org/2012/speaker/profile/390/>`_ (https://github.com/erikrose)
(`@ErikRose <https://twitter.com/#!/erikrose>`_)

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/473/

**Slides** (Keynote): https://github.com/downloads/erikrose/presentations/Speedily%20Practical%20Large-Scale%20Tests.key

**Slides** (HTML): http://msabramo.github.com/presentations/erik-rose/Speedily%20Practical%20Large-Scale%20Tests.html.d/Speedily%20Practical%20Large-Scale%20Tests.html

**Video**: http://pyvideo.org/video/634/speedily-practical-large-scale-tests


`iStat menus for the Mac <http://bjango.com/mac/istatmenus/>`_ so you know your
baseline memory usage, disk activity, etc.

The `Python profiler <http://docs.python.org/library/profile.html>`_ doesn't
tell you about I/O; only CPU. You can use the `UNIX time command
<http://en.wikipedia.org/wiki/Time_(Unix)>`_ to look at wall clock time and CPU
time and then do subtraction to
get an idea of I/O time. top, lsof

Conquest of Speed parts:

1. Per-class fixture loading
2. Fixture bundling
3. Startup Speedups

Got tests form 302 seconds down to 62 seconds.

`Nose <http://readthedocs.org/docs/nose/en/latest/>`_

* Finding and picking tests
* dealing with errors
* etc.

`django-nose <http://readthedocs.org/docs/nose/en/latest/>`_ 1.0 will be
released soon (Note: it's out - see `PyPI
<http://pypi.python.org/pypi/django-nose>`_) and Erik Rose will be sprinting on
it on Monday.

Die, setUp(), die
-----------------

More explicit test setup. Can be more efficient because you're not setting up stuff you don't need.

Die, fixtures, die
------------------

"model makers" -- d = document(title='test')

with_save decorator

`factory_boy <https://github.com/rbarrois/factory_boy>`_ for Python - based on
`thoughtbot <http://thoughtbot.com/>`_'s `factory_girl
<https://github.com/rbarrois/factory_boy>`_ for Rails

Shared setup makes tests...

* Coupled to each other
* Brittle
* Hard to understand
* Slow
* Kick puppies

Local setup gives you...

* Decoupling
* Robustness
* Clarity
* Efficiency
* Puppy kisses

Imperative vs. declarative

* `Mock <http://www.voidspace.org.uk/python/mock/>`_ - Imperative
* `Fudge <http://farmdev.com/projects/fudge/>`_ - Declarative

The `Mock <http://www.voidspace.org.uk/python/mock/>`_ Library::

    from mock import patch

    with patch.object(APIVoterManager, '_raw_voters') as voters:
        ....

The `Fudge <http://farmdev.com/projects/fudge/>`_ Library::

    @fudge.patch('sphinxapi.SphinxClient')
    def test_single_filter(sphinx_client):
        ....
        (sphinx_client.expects_call().returns_fake()
            .is_a_stub()
            .expects('SetFilter').with_args('a', [1], False)
            .expects('SetFilter')....
        ....

Horrible dots - doesn't tell you anything

Hate tracebacks - too much noise

Erik Rose put together an alternative nose test runner called `nose-progressive
<http://pypi.python.org/pypi/nose-progressive/>`_::

    % nosetests --with-progressive

* displays a progress bar instead of dots so you know how long tests might take
* much abbreviated tracebacks
* editor shortcuts with the + syntax for line numbers that you can copy and paste to edit the file

If you just want the improved tracebacks, check out `tracefront
<http://pypi.python.org/pypi/tracefront/0.1>`_ which is Erik Rose's extraction
of the traceback stuff from `nose-progressive
<http://pypi.python.org/pypi/nose-progressive/>`_.

How to install testing goodness::

    pip install nose-progressive
    pip install django-nose

`zope.testing <http://pypi.python.org/pypi/zope.testing>`_ package is pretty
well-decoupled from the rest of Zope and easy to use with non-zope stuff.  Also
Twisted's `Trial <http://twistedmatrix.com/trac/wiki/TwistedTrial>`_ test
runner - someone noted that it's nice but it doesn't work with Django because
Django is broken and wanted to sprint on it.

`David Cramer <http://justcramer.com/>`_ of `Disqus <http://disqus.com/>`_ --
`nose-bleed <http://pypi.python.org/pypi/nose-bleed>`_ and `nose-quickunit
<https://github.com/dcramer/nose-quickunit>`_

Someone mentioned that it's nice to set up editor keybindings that run the
tests for just the file you're editing. Another way is to have something like
`autonose <https://github.com/gfxmonk/autonose>`_ that automatically runs your
tests for you.

MySQL sucks at everything; wants to switch to Postgres at Votizen.
