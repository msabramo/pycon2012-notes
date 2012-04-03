********************************************************************************************************
Fake It Til You Make It: Unit Testing Patterns With Mocks And Fakes by Brian K. Jones
********************************************************************************************************

**Presenters**: `Brian K. Jones <https://us.pycon.org/2012/speaker/profile/275/>`_

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/336/

**Slides**: 

**Video**: https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video

**Video running time**: 49:33


Your Speaker
============

`(00:14) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=00m14s>`_

* `PSF <http://python.org/psf/>`_ member since 2011

* Creator: Python Magazine

* co-author: `Linux Server Hacks, vol. 2
  <https://www.amazon.com/dp/0596100825/ref=as_li_qf_sp_asin_til?tag=marcsepinion-20&camp=0&creative=0&linkCode=as1&creativeASIN=0596100825&adid=15318VSSWGSABJC9TKCF&>`_
  and upcoming `Python Cookbook 3rd edition
  <https://www.amazon.com/dp/1449378722/ref=as_li_qf_sp_asin_til?tag=marcsepinion-20&camp=0&creative=0&linkCode=as1&creativeASIN=1449378722&adid=1DAR4DJAN2TKGQXHP44Q&>`_

* `github.com/bkjones <https://github.com/bkjones>`_

* jonesy on freenode (join #python-testing!)

* `protocolostomy.com <http://www.protocolostomy.com/>`_


What's covered
==============

`(01:17) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=01m17s>`_

* Definitions

* Patterns

* Tools


What's not covered
==================

`(01:56) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=01m56s>`_

* Intro to the :mod:`unittest` module

* Sales pitch for doing testing at all


What is a "unit test"?
======================

`(02:38) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=02m38s>`_

* A test that exercises a very small amount of code which is completely isolated from any and all dependencies, external or internal

* Granular -- only testing a small piece of code

* Isolated

* Localized -- tells you exactly where your bug is


When it is no longer a unit test?
=================================

`(03:38) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=3m38s>`_

* When something else in the larger system, besides the code under test, has to work.


What is it then?
================

`(03:48) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=3m48s>`_

* If multiple components of the same class or system are tested, and the test
  seeks to determine that these components interact in a predictable way, it's
  an integration test.

* If the entire system is being tested to insure compliance with a spec of some
  kind, it's an acceptance test.


What is "coverage"?
===================

`(04:15) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=4m15s>`_

* A loaded term

* Often generically referred to as "code coverage".

* What you really want is "condition coverage", "case coverage", "logic
  coverage", "decision-point coverage", etc.


Use coverage.py
===============

`(05:15) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=5m15s>`_

http://nedbatchelder.com/code/coverage/

* Integrates well with `nosetests <http://readthedocs.org/docs/nose>`_

* Super easy to use

* Can report on branch coverage using ``--branch``

  - http://nedbatchelder.com/code/coverage/branch.html

* Can be used easily with `tox <http://tox.testrun.org/>`_, by itself
  or from within nosetests

* Nice work! Thanks, Ned!


Speaking of nosetests
=====================

`(07:04) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=7m04s>`_

* Nose is a great test discovery tool

* cd to your test directory and run "nosetests". Rejoice.

* ``nosetests --with-coverage`` (coverage.py is a nose plugin!)

* Has a super nice HTML report that highlights covered/uncovered lines

* Integrates well with Jenkins to present html and provide pretty graphs (managers like those):

  - http://pypi.python.org/pypi/nosexcover


Why unit tests?
===============

`(07:50) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=7m50s>`_

* They're fast

* They're simple

* They provide greater localization of problems than other types of tests often can


Unit tests aren't enough
========================

`(10:13) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=10m13s>`_

Unit tests don't test integration - that the parts of the system all work together


A problem
---------

`(10:54) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=10m54s>`_

(Sample Python code from `PyRabbit <https://github.com/bkjones/pyrabbit>`_)


A solution
----------

`(11:55) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=11m55s>`_

An integration test


`(12:05) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=12m05s>`_

`(12:26) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=12m26s>`_ "Reality distortion field"


Mock is cool. Use it.
=====================

`(12:40) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=12m40s>`_

* http://mock.readthedocs.org/en/latest/index.html

* It patches all the things

* Often used as a "spy" library (technically)

* I use it so I don't have to create my own mocking classes.

* Action->Assertion > Record->Replay

  - Action->Assertion is closer to how developers tend to think about their code


Mock handles harder stuff
-------------------------

`(13:54) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=13m54s>`_

...


Diagram
=======

`(15:10) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=15m10s>`_

...


More testable code
==================

`(17:00) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=17m00s>`_

Why there is resistance to adopting unit testing

Requires deeper knowledge of code

Requires special techniques

You have to invest time to learn it

A lot of managers, especially with large code bases, will not want to make that investment

However, unit testing will improve the design of your code.


Low-hanging fruit
-----------------

`(17:58) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=17m58s>`_

* Limit the scope of responsibility

* Create local wrappers

* Deduplicate the code


An example
----------

`(19:52) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=19m52s>`_

``get_path`` method

`(21:20) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=21m20s>`_

`(22:15) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=22m15s>`_


Practical patterns Part 1: A datetime abstraction library
=========================================================

`(23:27) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=23m27s>`_


Practical patterns Part 2: A REST client module
===============================================

`(28:54) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=28m54s>`_


pyrabbit
--------

* It's a cient for RabbitMQ's REST Management API
* ~250 lines of executable code
* ~200 lines of unit test code
* Uses :mod:`httplib2` to talk to the server
* Tests pass with Python 2.6, 2.7, and 3.2
* Use `tox <http://tox.testrun.org/>`_


tox is cool
-----------

`(30:06) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=30m06s>`_

http://tox.testrun.org/

.. code-block:: python

    [tox]
    envlist = py26,py27,py32

    [testenv]
    deps =
        nose
        httplib2
        mock
    changedir = tests
    commands = nosetests []

    [testenv:py26]
    deps =
        nose
        httplib2
        mock
        unittest2
    changedir = tests
    commands = nosetests []


Other tricks
============

`(38:19) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=38m19s>`_


Mocking stdout
--------------

`(38:27) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=38m27s>`_

.. code-block:: python

    outstream = StringIO()

    with patch('sys.stdout', new=outstream) as out:
        ...
        actual_out = out.getvalue()


Testing decorated functions
---------------------------

`(39:48) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=39m48s>`_


We've covered
=============

`(41:43) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=41m43s>`_

* What's a unit test? Why are they cool? How can I make some?

* Are unit tests all I need?

* What's `Mock <http://mock.readthedocs.org/en/latest/index.html>`_ ? Why is it
  cool? How can I use it?

* Use `tox <http://tox.testrun.org/>`_! Use `nosetests
  <http://readthedocs.org/docs/nose>`_! Use `coverage.py
  <http://nedbatchelder.com/code/coverage/>`_!

* Testing a simple one-module date manipulation library

* Testing a REST API client library

* And more!


Questions?
==========

`(42:12) <https://www.youtube.com/watch?v=hvPYuqzTPIk&list=PLBC82890EA0228306&index=13&feature=plpp_video#t=42m12s>`_

* `Mock <http://mock.readthedocs.org/en/latest/index.html>`_ is going to be in
  the Python Standard Library in Python 3.3 as :mod:`unittest.mock`.

* Question: How to organize integration tests?

* Question: When do you find doctests useful?

* Question: Design for testability (e.g.: dependency injection) vs. monkey-patching

  - Dependency injection is something that you need a team to be bought into

* Comment (from :ref:`Gary Bernhardt <fast-test-slow-test>`). Decorators make
  testing harder because they couple things together at compile-time. The
  solution is dependency injection.


