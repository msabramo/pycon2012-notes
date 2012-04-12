********************************************************************************************************
Practicing Continuous Deployment by David Cramer of DISQUS
********************************************************************************************************

**Presenters**: David Cramer from `DISQUS <http://disqus.com/>`_

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/12/

**Slides**:

**Video**: http://www.youtube.com/watch?v=QGfxLXoMpPk

**Video running time**: 41:20


What do we mean by continuous deployment?
=========================================

`(0:27) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=0m27s>`_

Shipping new code as soon as it's **ready**


What does "ready" mean?
=======================

`(0:54) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=0m54s>`_

* Reviewed by peers
* Passes automated tests -- continuous integration is essential
* Some level of QA

.. code-block:: bash

    ┌────────────────┐      ┌────────────────┐
    |     Review     |  ←   |     Commit     |
    └────────────────┘      └────────────────┘
            ↓                       ↑
    ┌────────────────┐      ┌────────────────┐
    |   Integration  |  →   |  Failed build  |
    └────────────────┘      └────────────────┘
            ↓
    ┌────────────────┐      ┌────────────────┐
    |    Deploy      |  →   |   Reporting    |
    └────────────────┘      └────────────────┘
                                    ↓
                            ┌────────────────┐
                            |    Rollback    |
                            └────────────────┘


`(02:43) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=2m43s>`_

**Continuous deployment does not necessarily mean that you deploy all the time or every 5 minutes.**

You can deploy as often as you want. The important thing is that you can deploy **whenever you want**.


The good and the bad
====================

`(3:14) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=3m14s>`_

The good
--------

* Develop features incrementally
* Release frequently
* Smaller doses of QA

The bad
-------

* Culture shock
* Stability depends on test coverage
* Initial time investment

At Disqus
---------

* `DISQUS <http://disqus.com/>`_ is a company of about 30 people.
* Spent the last 2 years working on infrastructure - automated testing tools, reporting, etc.
* We have a guy dedicated to tests and a guy dedicated to releasing.
* You really have to instill this in your culture.


Keep development simple
=======================

`(4:48) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=4m48s>`_

* **Automate testing** of complicated processes and architecture

* Simple can be better than complete

  - Especially for local development

* ``python setup.py {develop,test}``

* Puppet, Chef, Buildout, Fabric, etc.


**Automated testing is a requirement. Continuous integration is the basis for all of this.**

David feels that packaging your app is essential, because you need things to be repeatable.


Bootstrapping local
-------------------

`(7:32) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=7m32s>`_

* Simplify local setup

  - ``git clone dcramer@disqus:disqus.git``
  - ``make``
  - ``python manage.py runserver``

* Need to test dependencies?

  - virtualbox + ``vagrant up``    (`Link to Vagrant <http://vagrantup.com/>`_)


Progressive rollout
===================

`(9:01) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=9m01s>`_

We **actively use early versions of features** before public release.

At DISQUS, we do about 12,000 to 15,000 requests/second and we peak much higher than that.

It's important that a feature doesn't take the site down. We want to slowly release features.

`(9:32) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=9m32s>`_ Feature flippers or switches

**Deploy features to portions** of a user base at a time to ensure **smooth, measurable releases**

They use a platform called `Gargoyle <https://github.com/disqus/gargoyle>`_ --
currently very Django-specific, but trying to generalize it to be
Django-agnostic and maybe even language-agnostic.

Example:

1. Only enable this new feature for internal users.
2. OK, now turn it on for 1% of our base.
3. Keep bumping up until we know it's scalable.


Iterate quickly by hiding features
==================================

**Early adopters** are free QA

.. code-block:: python

    from gargoyle import gargoyle

    def my_view(request):
        if gargoyle.is_active('awesome', request):
            return 'new happy version :D'
        else:
            return 'old sad version :('

New users can check a box to volunteer to test bleeding edge features.


Review all the commits
======================

`(11:42) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=11m42s>`_

`Phabricator <http://phabricator.org/>`_ - a code review tool open-sourced by Facebook.

`(12:40) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=12m40s>`_ When you do a
code review, it's done through a commit - friendly for developers. Don't have
to use the Web UI.

``arc diff`` runs a set of lints and runs your tests for you.

They've released a plugin for nose called quickunit.


Integration == Jenkins
======================

`(15:10) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=15m10s>`_

Integration requirements
------------------------

`(15:45) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=15m45s>`_

* Developers *must* know when they've broken something

  - IRC, Email, IM

* Support proper reporting

  - XUnit, Pylint, Coverage.py

* Painless setup

  - ``apt-get install jenkins``


It's important for developers to know right away when stuff is broken so they
can ideally fix it before they've context switched to something else.


Integration issues
------------------

**False positives**

* Reporting isn't accurate
* Services fail (even a third party service)
* Bad tests

**Test coverage**

* Regressions on untested code

**Feedback delay**

* Integration tests vs. unit tests


Fixing false positives
----------------------

`(18:00) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=18m00s>`_

* Rerun tests several times on failure
* Report continually failing tests
* Replace external service tests with a functional test suite


Maintaining coverage
--------------------

`(18:38) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=18m38s>`_

* Raise awareness with reporting

  - Fail/alert when coverage drops on a build

* Commit tests with code

  - Coverage against commit diff for untested regressions

* Utilize code review


Speeding up tests
-----------------

`(20:05) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=20m05s>`_

This where almost all of our time has gone.

At one point our test suite took 40 minutes to an hour.

* Write unit tests

  - vs. slower integration tests

* Mock external services

* Distributed and parallel testing

  - Matrix builds


Reporting
=========

`(22:24) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=22m24s>`_

<You> Why is mongodb-1 down?

<Ops> It's down? Must have crashed again.

Meaningful metrics
------------------

* Rate of traffic (not just hits!)

  - Business vs. system

* Response time (database, web)

* Exceptions

* Social media

  - Twitter


Tools
-----

`(24:18) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=24m18s>`_

Beyond Nagios and PagerDuty.

Graphite
--------

Tracks and graphs metrics

`graphite.wikidot.com <http://graphite.wikidot.com/>`_

We send it response times, counters, disk space usage


Sentry
------

https://www.getsentry.com/

You can now use it even if you're not using Django.

It's designed to receive exceptions and track them.


Wrap up
=======

`(26:08) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=26m08s>`_

**Deployment** - the least important part of continuous deployment. Everyone solves it differently.

What DISQUS does. Ship a relocatable virtualenv as a tarball.


Getting Started
---------------

`(27:02) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=27m02s>`_

* Package your app
* Value code review
* Ease deployment, **fast rollbacks**
* Setup automated tests
* Gather some easy metrics


Going further
-------------

`(29:00) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=29m00s>`_

* Build an immune system -- automatically rolls back if some metric goes down -- very interesting, but very risky

  - Automate deploys, rollbacks (maybe)

* Adjust to your culture

  - There is no "right way"

* SOA == great success


Questions?
==========

`(30:25) <http://www.youtube.com/watch?v=QGfxLXoMpPk#t=30m25s>`_

Code reviews: Before Phabricator, DISQUS used GitHub pull requests but they found it to not be scalable.

(31:49) Selenium tests -- we deleted all our Selenium tests. We're reimplementing some of them, but simpler.

(32:50) How many times a day do you deploy? At minimum, once a day. Lately, it's been no more than half a dozen times per day.

(33:55) Why do you roll back? Why not fix it and move forward? Sometimes it might take a while to fix it.

(34:30) What do you do about database changes? Especially for rollbacks. Google DISQUS schema changes or David Cramer schema changes

(35:52) Any code review policies? Maximum # of lines or maximum amount of time
until review. Current standard is at the start of the day and the end of the
day, you must clean your slate. Even this kind of sucks, because you may have
to wait a day to get your change reviewed. What we really want is to give a max
of 20 minutes and if it isn't reviewed, then it automatically gets assigned to
someone else.

(37:23) Numbers of production servers? 200ish. 4 billion pageviews.

(38:00) How long does it take to deploy? Ashamed to admit it. All of our servers in one location although we push a lot of stuff to Akamai.

(39:00) One monolithic deploy or many? We're moving towards SOA and away from monolithic.

(40:15) Can you tell us about your rollback process? At one point, it was just swap the symlink and restart the servers.

(40:33) Business metrics measurements - what tools? Graphite, statsd, porkchop

(41:10) Done.

