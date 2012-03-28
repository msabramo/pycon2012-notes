***************************************************************************
Scalability at YouTube
***************************************************************************

**Presenters**:  J.J. Behrens and Mike Solomon of Google

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/315/

**Slides**: ???

**Video**: https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video

**Video running time**: 38:43


First few minutes of the video are marketing fluff.

`(10:16)
<https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=10m16s>`_
The talk really starts.


Briefly
=======

* scalability
* efficiency
* productivity

People often confuse scalabity and efficiency.


Scalable...systems
==================

`(12:32) <https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=12m32s>`_

* simple - "If I could give you one word about scalable systems, I think the word would be *simple*"
* solve the problem at hand
* product of evolution


The Tao of Youtube
==================

"choose the simplest solution with the loosest guarantees that are practical"


Scalable...techniques
=====================

`(14:51) <https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=14m51s>`_

* divide and conquer
  - e.g.: independent web servers, database sharding, partition the problem, simple and loose connections
* approximate correctness - cheat and let people see their own comments rather than requiring immediate consistency
* expert knob twiddling
  - consistency, durability
* jitter - introduce more randomness, because surprisingly, things tend to stack up
  - example: cache expirations - if all caches expires at the same time => thundering herd. Add some random variance.
* cheat

Jitter puts entropy back into the system.


Scalable...components
=====================

`(21:06) <https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=21m06s>`_

* well defined inputs
* well defined dependencies
* frequently end up behind an RPC
* leverage local optimizations


Scalable...humans?
==================

`(23:12) <https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=23m12s>`_

* communication through code/data/schema
* RPC as a means of sanity


Efficiency
==========

`(24:26) <https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=24m26s>`_

* uncorrelated with scalability
* focus on algorithms first
* learn to measure
  - use representative samples


Efficient libraries
===================

`(26:40) <https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=26m40s>`_

* `wiseguy <http://code.google.com/p/msolo/wiki/wiseguy>`_ - YouTube's fast CGI container
* `pycurl <http://pycurl.sourceforge.net/>`_
* `spitfire <http://code.google.com/p/spitfire/>`_ - YouTube's templating system; similar to Cheetah
* serialization formats* - Don't use pickle


Efficient tools
===============

`(29:46) <https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=29m46s>`_

* apache
* linux
* mysql
* `vitess <http://code.google.com/p/vitess/>`_
* `zookeeper <http://zookeeper.apache.org/>`_


Productivity
============

`(32:12) <https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=32m12s>`_

* philosophy vs. doctrine
* more conventions
  - less documentation
* more effective collaboration
* more diffusion of responsibility


Zen proverb
===========

`(33:54) <https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=33m54s>`_

"Do not seek to follow in the footsteps of masters; seek what they sought"


Code Bonsai
===========

`(34:22) <https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=34m22s>`_

* simple
* pragmatic
* elegant
* orthogonal
* composable


Questions?
==========

`(35:00) <https://www.youtube.com/watch?v=G-lGCC4KKok&list=PLBC82890EA0228306&index=22&feature=plpp_video#t=35m00s>`_




