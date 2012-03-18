.. _fast-test-slow-test:

***************************************************************************
Fast test, slow test
***************************************************************************

**Presenter**: `Gary Bernhardt
<https://us.pycon.org/2012/speaker/profile/366/>`_ (http://blog.extracheese.org/ / https://www.destroyallsoftware.com/)
(`@garybernhardt <http://twitter.com/garybernhardt>`_)

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/429/

**Slides**: ???

**Video**: http://pyvideo.org/video/631/fast-test-slow-test


Writing tests from the bottom up is often easier

.. code-block:: python

    def test_that_spam_posts_are_hidden(self):
        assert 'Spammy!" not in resp.content

Note that this is a negative assertion, which is dangerous.

The binary test suite - tells you whether or not your code is broken but gives
no clues to what. Good tests show you exactly what's broken.

Fine-grained tests around legcy code -- a way to fail -- bakes all of the
badness in. :-)

Should you mock the model objects in a Django app? No it's too wide of a
boundary. A better approach is to create a service layer that interacts with
the model objects and then mock that service layer.

Gary Bernhart has a mocking module called `Dingus
<http://pypi.python.org/pypi/dingus>`_.

He mostly works in Ruby these days and uses `Cucumber <http://cukes.info/>`_
with `Capybara <http://jnicklas.github.com/capybara/>`_ driving a headless
WebKit browser.

`Mock by Michael Foord <http://www.voidspace.org.uk/python/mock/>`_ does interface checks.
