.. _fast-test-slow-test:

***************************************************************************
Fast test, slow test by Gary Bernhardt from destroyallsoftware.com
***************************************************************************

**Presenter**: `Gary Bernhardt
<https://us.pycon.org/2012/speaker/profile/366/>`_ (http://blog.extracheese.org/ / https://www.destroyallsoftware.com/)
(`@garybernhardt <http://twitter.com/garybernhardt>`_)

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/429/

**Slides**: ???

**Video**: http://pyvideo.org/video/631/fast-test-slow-test

**Video running time**: 31:50


Goals of tests
==============

1. Prevent regressions

The weakest of the goals. Doesn't change the way you build the software minute
to minute. At best it changes how you release the software. You don't release
broken things.

2. Prevent fear `(00:19)
   <https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=00m19s>`_

Prevent fear minute to minute or second to second, so speed is important here. Enable refactoring.

3. Prevent bad design `(00:52)
   <https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=00m52s>`_

Very subtle topic. Sort of out of bounds. The holy grail of testing.

`(01:32)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=01m32s>`_
Video of a large system test running.

`(02:06)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=02m6s>`_
A smaller, synthetic example to dissect for this talk -- test for a Django app; a discussion board

Writing tests from the bottom up is often easier

.. code-block:: python

    def test_that_spam_posts_are_hidden(self):
        assert 'Spammy!" not in resp.content

.. raw:: html

    <div style="margin-left: 150px">↓</div>

.. code-block:: python

    def test_that_spam_posts_are_hidden(self):
        resp = self.client.get("/discussion/%s" % disc.pk)
        assert 'Spammy!" not in resp.content

.. raw:: html

    <div style="margin-left: 150px">↓</div>

.. code-block:: python

    def test_that_spam_posts_are_hidden(self):
        self.client.post("/mark_post_as_spam", {'post_id': post.pk})
        resp = self.client.get("/discussion/%s" % disc.pk)
        assert 'Spammy!" not in resp.content

.. raw:: html

    <div style="margin-left: 150px">↓</div>

.. code-block:: python

    def test_that_spam_posts_are_hidden(self):
        log_in(alice)
        self.client.post("/mark_post_as_spam", {'post_id': post.pk})
        resp = self.client.get("/discussion/%s" % disc.pk)
        assert 'Spammy!" not in resp.content

.. raw:: html

    <div style="margin-left: 150px">↓</div>

.. code-block:: python

    def test_that_spam_posts_are_hidden(self):

        disc = Discussion()

        disc.save()
        log_in(alice)
        self.client.post("/mark_post_as_spam", {'post_id': post.pk})
        resp = self.client.get("/discussion/%s" % disc.pk)
        assert 'Spammy!" not in resp.content

.. raw:: html

    <div style="margin-left: 150px">↓</div>

.. code-block:: python

    def test_that_spam_posts_are_hidden(self):

        disc = Discussion()
        disc.posts.append(Post(poster=bob, "Spammy!"))
        disc.save()
        log_in(alice)
        self.client.post("/mark_post_as_spam", {'post_id': post.pk})
        resp = self.client.get("/discussion/%s" % disc.pk)
        assert 'Spammy!" not in resp.content

.. raw:: html

    <div style="margin-left: 150px">↓</div>

.. code-block:: python

    def test_that_spam_posts_are_hidden(self):
        alice, bob = User(admin=True), User(admin=False)
        disc = Discussion()
        disc.posts.append(Post(poster=bob, "Spammy!"))
        disc.save()
        log_in(alice)
        self.client.post("/mark_post_as_spam", {'post_id': post.pk})
        resp = self.client.get("/discussion/%s" % disc.pk)
        assert 'Spammy!" not in resp.content

`(03:47)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=03m47s>`_
Why is this a system test?

What does it depend on? What thing in this test could cause it to break?

.. code-block:: python

    def test_that_spam_posts_are_hidden(self):
        alice, bob = User(admin=True), User(admin=False)
                          FLAG
        disc = Discussion()
                          SIGNATURE
        disc.posts.append(Post(poster=bob, "Spammy!"))
             RELATIONSHIP      SIGNATURE
        disc.save()
             VALIDITY
        log_in(alice)
        AUTH
        self.client.post("/mark_post_as_spam", {'post_id': post.pk})
                         URL, SIGNATURE, PRECONDITIONS
        resp = self.client.get("/discussion/%s" % disc.pk)
                           URL, SIGNATURE, PRECONDITIONS
        assert 'Spammy!" not in resp.content
               REPRESENTATION (E.G., NOT AJAX)


Note that:

.. code-block:: python

    assert 'Spammy!" not in resp.content

is a negative assertion, which is dangerous. If we change the view to render a
skeleton and then fill in the details later with AJAX requests, this assertion
*will always succeed*. This is one of the dangers of negative assertions.

`(05:59)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=05m59s>`_
We are also dependent on:

* Django test client
* Django router
* Django request object
* Django response object
* Third party middleware (!!!)
* App middleware (!!!)
* Context managers (!!!)

`(06:41)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=06m41s>`_
The result of these dependencies is that we end up with a *binary test suite* -
tells you whether or not your code is broken but gives no clues to what. Good
tests show you exactly what's broken.

`(07:10)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=07m10s>`_
*Test fragility* -- "Every time we change the code, we have to update all the tests!"

`(07:32) <https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=07m32s>`_
We primarily get regression protection (and only specific kinds; the layers integrating incorrectly)

It's very difficult to test fine-grained edge cases from the outside.

It's not fast so it won't help with refactoring.

No feedback on design since we're not interacting with the smaller objects.

System tests have value but also have problems.


How To Fail
===========

`(08:29)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=08m29s>`_
3 ways to fail:

1. Selenium as primary testing tool

2. "Unit tests" are too big

3. Fine-grained tests around legacy code


Selenium as primary testing
---------------------------

`(08:34) <https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=08m29s>`_

* Tests can't be run locally
* Tests too slow
* Tests break often
* No fine-grained feedback


"Unit tests" are too big
------------------------

`(09:20) <https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=09m20s>`_

Testing time tends to grow super-linearly.

100ms = 240,000,000 instructions


Fine-grained tests around legacy code
-------------------------------------

`(10:44)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=09m20s>`_
Fine-grained tests around legcy code -- a way to fail -- tight tests solidify the
interface and bake all of the badness in. :-)


Unit tests
==========

`(11:20)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=11m20s>`_
Unit tests - what are they and why do we care?

Showed two videos of very fast test suites.

`(12:29)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=12m29s>`_
We will test at the *model layer* instead of the view layer.

.. code-block:: python

    def test_that_spam_posts_are_hidden(self):
        post = Post(mark_post_as_spam=True)
        discussion = Discussion(posts=[post])
        assert discussion.visible_posts == []

This is a complete test at the model layer. This does not replace system tests;
does not test views.

`(13:18)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=13m18s>`_
Why is this a unit test?

`(13:26)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=13m26s>`_
1. Unit tests test only *one object behavior*.

`(13:59)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=13m59s>`_
2. Other classes can't break it.

And in particular, no dependencies on:

* Django test client
* Django router
* Django request object
* Django response object
* Third party middleware (!!!)
* App middleware (!!!)
* Context managers (!!!)


What are the advantages of unit tests?
--------------------------------------

`(15:12)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=15m12s>`_
Test failures are much more isolated and tell you which object or method is broken.

`(15:26)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=15m26s>`_
Tests are much faster. You can avoid fear. You can refactor. The difference
between 400 milliseconds and 40 seconds -- at 40 seconds, you can't do the
thing called TDD.

`(15:40)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=15m40s>`_
System tests test the boundaries better than unit tests. Unit tests test the
fine-grained behavior of individual objects, which is most of the behavior of
your system, hopefully.

`(16:10)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=16m10s>`_
Unit tests enable refactoring and let you avoid fear.

`(16:24)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=16m24s>`_
*Gives you design feedback*. I have conditioned myself to be repulsed by an 8
line test for a model. Why do I need to set up so much of the world to test
this one small piece of behavior? Makes me think about refactoring, which leads
to better system design.

`(16:46)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=16m46s>`_
Guidelines for ratio of unit tests to system tests -- 90% unit tests, 10% system/acceptance tests

`(17:19)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=17m19s>`_
I have not mentioned test doubles or mocking. You may not these when testing
the low levels like models. You may need them when testing higher level objects
like views.


The End
=======

`(18:00) <https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=18m00s>`_

@garybernhardt

destroyallsoftware.com

Screencasts for Serious Developers

* OO design
* Unix
* TDD
* Smaller, faster tests


Questions
=========

`(18:26)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=18m26s>`_
Q: 90/10 ratio - was that in time or lines of code or what?

A: Number of tests

`(18:50)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=18m26s>`_
Q: Does 90/10 apply to every kind of project or does it vary?

`(19:04)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=18m26s>`_
A: That applies mostly to object-heavy systems like web apps, that have lots of logic and not a lot of
boundaries.

`(19:44)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=19m44s>`_
Question (from Carl Meyer): Pain point in unit testing Django apps is the
database. Slows down your tests. Django models objects are very tied to the
database. Trying to mock out the persistence layer seems like a way to fail.

`(20:36) <https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=20m36s>`_ Answer:
Should you mock the model objects in a Django app? No it's too wide of a
boundary that you don't control. A better approach is to create a service layer
that interacts with the model objects and then mock that service layer.

`(22:00)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=22m00s>`_
Question: How do you enforce that mock objects have the same behavior as the real object?

`(22:19)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=22m19s>`_
Answer: System tests. Or in Ruby, `rspec-fire from Xavier Shay
<https://github.com/xaviershay/rspec-fire>`_

`(23:36)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=23m36s>`_
`Mock by Michael Foord <http://www.voidspace.org.uk/python/mock/>`_ does
interface checks.

`(24:32)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=24m32s>`_
Question: Why is it such a bad idea to unit test legacy code?

`(24:37)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=24m37s>`_
Answer: It is good to test unit test legacy code. It's not good to write
*fine-grained tests* for legacy code, because it solidifies the edges.  I may
be the worst offender, because my mocking library `Dingus
<http://pypi.python.org/pypi/dingus>`_ can magically mock everything on the
outside layer of your class, so if you're doing this, stop it. :-)
You want to read "Working Effectively With Legacy Code" by Michael Feathers.

`(26:05)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=26m05s>`_
Integration tests == system tests?

`(26:09)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=26m09s>`_
Answer: Oops, sorry. Main distinct is unit test (which tests one thing) vs.
any kind of integrated test that tests multiple things.

`(27:39)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=27m39s>`_
Question: When is Selenium appropriate?

`(27:55)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=27m55s>`_
Answer: Selenium is not evil. If you use Selenium to test everything and
especially fine-grained behavior, that's where you'll run into problems.

`(28:20)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=28m20s>`_
He mostly works in Ruby these days and uses `Cucumber <http://cukes.info/>`_
with `Capybara <http://jnicklas.github.com/capybara/>`_ driving a headless
WebKit browser.

`(28:38)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=28m38s>`_
Don't pay non-programmers to build large Selenium test suites.

`(29:15)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=29m15s>`_
Question: How do I convert a system test suite to unit tests and make sure that
I'm covering everything the system tests covered?

`(29:25)
<https://www.youtube.com/watch?v=RAxiiRPHS9k&list=PLBC82890EA0228306&index=3&feature=plpp_video#t=29m25s>`_
Answer: Ask Michael Feathers? :-) Sometimes it's obvious...

