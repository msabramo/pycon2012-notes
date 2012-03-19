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

