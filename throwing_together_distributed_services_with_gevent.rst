***************************************************************************
Throwing Together Distributed Services With Gevent
***************************************************************************

**Presenter**: `Jeff Lindsay
<https://us.pycon.org/2012/speaker/profile/238/>`_ (http://progrium.com/)
(`@progrium <https://twitter.com/#!/progrium>`_)

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/288/

**Tutorial**: https://github.com/progrium/ginkgotutorial

**Video**: http://pyvideo.org/video/642/throwing-together-distributed-services-with-geven

**Track:** V

Description

In this talk we learn how to throw together a distributed system using `gevent
<http://www.gevent.org/>`_ and a simple framework called `Ginkgo
<https://github.com/progrium/ginkgo>`_ (formerly `gservice
<https://github.com/progrium/gservice>`_). We'll go from nothing to a
distributed messaging system ready for production deployment based on
experiences building scalable, distributed systems at `Twilio
<http://www.twilio.com/>`_.

Abstract

As some have found, `gevent <http://www.gevent.org/>`_ is one of the best kept
secrets of Python. It gives you fast, evented network programming without
messes of callbacks, code that is more Pythonic, and lets you use most regular
Python networking libraries and protocol implementations. Now, let's build on
this.

In this talk we learn how to throw together distributed services using `gevent
<http://www.gevent.org/>`_ and a simple framework called `Ginkgo
<https://github.com/progrium/ginkgo>`_ (formerly `gservice
<https://github.com/progrium/gservice>`_). We'll go from nothing to a
distributed messaging system based on experiences building scalable,
distributed systems at `Twilio <http://www.twilio.com/>`_.

This talk will be full of code, live coding, and real production applications
with guest appearances by other fun technologies like `ZeroMQ
<http://www.zeromq.org/>`_, `WebSocket
<http://en.wikipedia.org/wiki/WebSocket>`_, and `Doozer <https://github.com/ha/doozerd>`_.

Jeff works at `Twilio <http://www.twilio.com/>`_. Twilio uses a service
oriented architecture. Twilio as a high level service is made up of many
subservices - sms, voice, client.  Each is made up of other services. Mostly
written in `tornado <http://www.tornadoweb.org/>`_ and `gevent
<http://www.gevent.org/>`_.  Going to use a framework called `Ginkgo
<https://github.com/progrium/ginkgo>`_.

* services are nested modules that can start, stop and reload
* going to build a scalable gateway around a self-organizing messaging system.
* first we'll build a simple number client
* next a pubsub service
* then combine both into a gateway service
* and finally make it all distributed

Number Server
-------------

* a basic `gevent StreamServer <http://www.gevent.org/gevent.server.html>`_
* for every connection generate a random number, sleep for 60 seconds
* configuration is a python file
* ginkgo has start / stop services
* start the number server in the background

Number Client
-------------

* spawns a `greenlet <http://pypi.python.org/pypi/greenlet>`_ relative to your service
* makes services self-contained
* gets the numbers from the server and puts it in a queue

PUBSub
------

* uses httpstreamer
* subscription wraps a queue
* posts are publishes, gets are subscribes

MessageHub
----------

* hub is now responsible for managing subscriptions

If you're in a loop that doesn't use IO, run ``gevent.sleep(0)`` to make sure it yields.

Code for the example is up here: https://github.com/progrium/ginkgotutorial


* Ginkgo - https://github.com/progrium/ginkgo
* Ginkgo tutorial - https://github.com/progrium/ginkgotutorial
* Notes from Andrew Schoen - http://readthedocs.org/docs/andrew-schoen-pycon-2012-notes/en/latest/friday/session_5.html
