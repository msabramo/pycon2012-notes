***************************************************************************
Testing and Django
***************************************************************************

**Presenter**: `Carl J. Meyer
<https://us.pycon.org/2012/speaker/profile/170/>`_ (http://oddbird.net/)
(`@carljm <https://twitter.com/#!/carljm>`_)

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/412/

**Slides**: http://carljm.github.com/django-testing-slides/

**Video**: http://pyvideo.org/video/699/testing-and-django

**Video running time**: 47:15


The presenter
=============

**Name**: Carl J. Meyer

**Email**: `carl@oddbird.net <mailto:carl%40oddbird.net>`_

**Slides**: https://github.com/carljm/django-testing-slides  [ `View slides
<http://carljm.github.com/django-testing-slides/>`_ ] (The slides use `Scott
Chacon <https://github.com/schacon>`_'s `ShowOff
<https://github.com/schacon/showoff>`_)

**Code from slides**: https://github.com/carljm/django-testing-slides/code


Upfront
=======

(00:30 / 47:15 into the video)

* The slides are online. The code is online.

* There are going to be a lot of opinions in this talk about better ways to do
  testing.

* Not a certified expert.

* Opinions do not reflect the opinions of the Django developers.

* We'll be using some features in the new Django 1.4 release candidate (`Django
  1.4 release notes <https://docs.djangoproject.com/en/dev/releases/1.4/>`_).


Let's start a project
=====================

(01:18 / 47:15 into the video)

.. code-block:: bash

    $ django-admin.py startproject testing .
    ## The period is a new Django 1.4 feature: the ability to start a new
    ## project in the current directory instead of creating a new subdirectory.
    ## (see https://code.djangoproject.com/ticket/17042)

    $ sed -i -e 's/backends\./backends.sqlite3/;' testing/settings.py
    ## Don't normally edit files using sed; it goes on a slide better than
    ## opening up an editor. Setting database backend to sqlite3.

    $ ./manage.py test
    ...
    [snip]
    ----------------------------------------------------------------------
    Ran 412 tests in 14.235s

    FAILED (errors=2, skipped=1)
    Destroying test database for alias 'default'...

This is the default Django testing experience. That ran **412** tests and took
**14** seconds to do it. And that's using an in-memory sqlite database, which
is the *fastest* way you can run Django's tests.

What's wrong here?

* We'll never get those 14 seconds back. That's a long time to wait, given we
  haven't started writing code!

* 2 failures in Django 1.4 RC - Those are due to a bug in Django that two
  tests in :mod:`django.contrib.auth` are insufficiently isolated from the project
  settings. They assume the existence of two databases configured in the
  settings.

Note from Marc on 2012-03-17: I just downloaded `Django-1.4c2
<http://www.djangoproject.com/download/1.4-rc-2/tarball/>`_ and tried to
reproduce the 2 failures, but I couldn't. So it looks like this issue was
fixed.

Not all apps are created equal
------------------------------

(03:26 / 47:15 into the video)

There is a difference between apps that **you** create in your project and apps
built into Django or third-party apps that you install.

For third-party apps, the only times I care about their tests is when I decide
to use them or upgrade them. In between those times, I don't care and running
their tests is a waste of time.

Some might argue that we need integration tests to verify that projects
integrate properly with things like :mod:`django.contrib.auth`, but any tests
written in :mod:`django.contrib.auth` that are not isolated are likely to fail,
because there are so many ways to integrate. And an isolated test is probably
just purely testing :mod:`django.contrib.auth` and projects shouldn't have to
care about that.  In other words...

* Non-isolated tests break.
* Isolated tests are pointless to run.
* Integration tests should be written by the integrator.

This isn't a big problem. You can just do this:

.. code-block:: bash

    ./manage.py test just my apps please

You could even wrap the above up in a shell script and you're done.

There's another problem...

:file:`tests/__init__.py`
-------------------------

(05:32 / 47:15 into the video)

.. code-block:: python

    from .test_forms import QuoteFormTest
    from .test_models import (
        QuoteTest, SourceTest)
    from .test_views import (
        AddQuoteTest, EditQuoteTest,
        ListQuotesTest
        )

Django insists that all of your tests live in a :file:`tests` module for each
app.

If you split your tests into a bunch of separate modules (which you probably
should if you're writing as many tests as you should), you have to import your
submodules so :ref:`Django's test runner
<topics-testing-test_runner>` can find it. This is 2012. That's
ridiculous.

Django's test discovery
-----------------------

(06:10 / 47:15 into the video)

* Wastes my time with tests I don't care about.
* Requires app tests to be in a single module (resulting in boilerplate imports).
* Forces intermingling of tests and non-test code.

It's easy to change.
--------------------

(06:50 / 47:15 into the video)

* :ref:`unittest2 test discovery <unittest-test-discovery>` (You could also use
  `nose <http://readthedocs.org/docs/nose/>`_, `py.test <http://pytest.org/>`_,
  etc...)

* ``TEST_RUNNER`` setting

This is how much code it takes to make Django's test discovery good:

.. code-block:: python

    class DiscoveryRunner(DjangoTestSuiteRunner):
        """A test suite runner using unittest2 discovery."""

        def build_suite(self, test_labels, extra_tests=None, **kwargs):
            suite = None
            discovery_root = settings.TEST_DISCOVERY_ROOT

            if test_labels:
                suite = defaultTestLoader.loadTestsFromNames(
                    test_labels)

            if suite is None:
                suite = defaultTestLoader.discover(
                    discovery_root,
                    top_level_dir=settings.BASE_PATH,
                    )

            if extra_tests:
                for test in extra_tests:
                    suite.addTest(test)

            return reorder_suite(suite, (TestCase,))

:file:`settings.py`:

.. code-block:: python

    import os.path
    BASE_PATH = os.path.dirname(os.path.dirname(__file__))
    TEST_DISCOVERY_ROOT = os.path.join(BASE_PATH, "tests")
    TEST_RUNNER = "tests.runner.DiscoveryRunner"

(There's an `updated version <https://gist.github.com/1450104>`_ of this test runner.)

\\o/
----

(07:59 / 47:15 into the video)

* Discovers tests wherever you want them.
* Doesn't run tests from external apps by default.
* Flexible specification of specific tests to run: Python dotted path to test
  module, not Django app label.
* ``./manage.py test tests.quotes.test_views``

Maybe in 1.5?
-------------

(08:37 / 47:15 into the video)

* https://code.djangoproject.com/ticket/17365

Types of test
-------------

(08:59 / 47:15 into the video)

* "Much inferior restatement of :ref:`Gary Bernhardt's excellent 'Fast Test, Slow Test' talk <fast-test-slow-test>` from yesterday"
* unit
* system/integration/functional

Unit tests
----------

(09:34 / 47:15 into the video)

* Test one unit of code (a function or method) in something approaching isolation.
* Fast, focused (useful failures).
* Help you structure your code better.

TBD: Add some more detail here on what Carl said.

Integration tests
-----------------

(10:35 / 47:15 into the video)

* Also very important.
* Test that the whole system works; catch regressions.
* Slow.
* Less useful failures. (Tell you something is broken, but takes longer to
  debug because it doesn't tell you where the problem is)
* Write fewer. Most people have too many of these. Django makes these easy with
  Django test client, but you shouldn't have too many of these.

Testing models
==============

(11:46 / 47:15 into the video)

The database makes your tests slow.
-----------------------------------

* Try to write tests that don't hit it at all. (:ref:`Erik Rose's talk <speedily-practical-large-scale-tests>` had an
  `excellent slide
  <http://erikrose.github.com/presentations/speedily-practical-large-scale-tests/Speedily%20Practical%20Large-Scale%20Tests.html_files/Speedily%20Practical%20Large-Scale%20Tests.017.png>`_
  about the latency of L1 cache vs. L2 cache vs. memory vs.
  disk -- disk accesses are **super** expensive)

* Separate db-independent model-layer functionality from db-dependent functionality.

  - Django doesn't make it easy to write tests that avoid hitting the database
    because the model layer encourages you to tie your models to the database.

  - So you need to do a bit of work here.

* But you'll still have a lot of tests that do.

* Mocking the database usually isn't worth it.

  - This is bound to fail. It's not a small and well-defined API so it's a lot of work.


A simple example of refactoring a model to separate out the db-independent functionality
----------------------------------------------------------------------------------------

(13:57 / 47:15 into the video)

Before:

.. code-block:: python

    class Thing(models.Model):
      def frobnicate(self):
        """Frobnicate and save the thing."""
        # ... do something complicated
        self.save()

There may be 20 different code paths before ``self.save()``. If we test all of
them, all of those tests will hit the database.

After:

.. code-block:: python

    def frobnicate_thing(thing):
      # ... do something complicated
      return thing

    class Thing(models.Model):
      def frobnicate(self):
        """Frobnicate and save the thing."""
        frobnicate_thing(self)
        self.save()

Pull out all the code that does the state modification and complex logic and
make it not talk to the database. And then **one** test that tests that it
saves to the database.


django.test.TestCase
--------------------

(15:22 / 47:15 into the video)

* Here come some boring slides. Don't have any problem with how Django does
  this stuff...

* Runs each test within a transaction.
* Rolls back the transaction at the end of the test.
* Monkeypatches transaction functions in your code to be no-ops.

This is nice because you don't have to have your tests truncate database tables
or recreate database state.

TransactionTestCase
-------------------

(15:58 / 47:15 into the video)

* Lets you test transactions in your code (doesn't wrap your tests in a transaction).
* Hash to flush every database table after every test.
* Makes your tests extra super bonus slow.
* You want to have as few of these as possible.

Fixtures
--------

(16:25 / 47:15 into the video)

* Set up database state ahead of time.

* Currently, the Django documentation points you to do these with **fixtures**
  (see `"Providing initial data for models"
  <https://docs.djangoproject.com/en/dev/howto/initial-data/>`_).

* Example JSON fixture:

.. code-block:: javascript

    [
        {
            "pk": 4,
            "model": "auth.user",
            "fields": {
                "username": "manager",
                "first_name": "",
                "last_name": "",
                "is_active": true,
                "is_superuser": false,
                "is_staff": false,
                "last_login": "2012-02-06 15:06:44",
                ...
            }
        },
        ...
    ]

* **Don't do it.** If you've got them in your code, **burn them**.


Fixtures: Just say no.
----------------------

(16:47 / 47:15 into the video)

[Applause :-)]

Probably the third talk that said this.

* Hard to maintain and update.

  - hand editing JSON => terrible

  - changing stuff in the db and then dumping them => terrible

  - if you're clever you can use the `django-fixture-generator app
    <https://github.com/alex/django-fixture-generator>`_ but might as well just
    skip fixtures altogether.

* Increase test interdependence

  - too tempting to just throw things in the fixture and then the shared
    fixture couples tests together => **unnecessary coupling** between tests.

* Slow to load.

  - People tend to put too much stuff in fixtures and every test loads the
    fixture and incurs the cost. There are tricks that Erik Rose talked about
    yesterday which are useful for legacy code (e.g.: "Per-Class Fixtures", but
    if you're writing new code, just don't use fixtures.


Model factories!
----------------

(18:41 / 47:15 into the video)

It's hard to use just vanilla ORM to set up dependencies because models have
dependencies so you end up writing a lot of stuff just to get one model to
test. This is why people like fixtures. But model factories are a better
solution.

An example of something that you could write yourself, with no special tools: a
user profile model.

.. code-block:: python

    def create_profile(**kwargs):
        defaults = {
            "likes_cheese": True,
            "age": 32,
            "address": "3815 Brookside Dr",
        }
        defaults.update(kwargs)
        if "user" not in defaults:
            defaults["user"] = create_user()
        return Profile.objects.create(
            **defaults)


Using a factory
---------------

(20:02 / 47:15 into the video)

.. code-block:: python

    def test_can_vote(self):
        """A user 18 age+ can vote in the US."""
        profile = create_profile(age=18)
        self.assertTrue(profile.can_vote)


Or use factory_boy:
-------------------

(21:22 / 47:15 into the video)

`factory_boy <https://github.com/rbarrois/factory_boy>`_

(a clone of Ruby's `factory_girl
<https://github.com/thoughtbot/factory_girl>`_)

.. code-block:: python

    class ProfileFactory(factory.Factory):
        FACTORY_FOR = Profile

        likes_cheese = True
        age = 32
        address = "3815 Brookside Dr"
        user = factory.SubFactory(UserFactory)

    profile = ProfileFactory.create(
        age=18, user__username="carljm")

* ``.create()`` saves the model object to the database.
* ``.build()`` builds the model object but doesn't save it to the database.
* So you can be explicit in your tests about whether or not they require the database.


Why factories?
--------------

(22:24 / 47:15 into the video)

* Test data local to test code (explict).

  - Makes test clearer and easier to maintain. Nothing depending upon some
    distant fixture.

* Easy to maintain.

* Don't create any data you don't need for that test.

* Works great even for large/complex test data sets (helper functions).


Imposing no-DB discipline.
--------------------------

(23:22 / 47:15 into the video)

* Django makes it easy to not be clear whether or not you're talking to the database.

* It can be helpful to specify that a test doesn't hit the database and have my
  code yell at me if it does.

* For certain test cases.

(23:49 / 47:15 into the video)

Solution: Use `Michael Foord's Mock library
<http://www.voidspace.org.uk/python/mock/>`_...

(Carl uses this library in every project. It is really useful for
monkeypatching things for the sake of testing).

.. code-block:: python

    from django.utils.unittest import TestCase
    import mock

    cursor_wrapper = mock.Mock()
    cursor_wrapper.side_effect = \
        RuntimeError("No touching the database!")

    @mock.patch(
        "django.db.backends.util.CursorWrapper",
        cursor_wrapper)
    class NoDBTestCase(TestCase):
        """Will blow up if you database."""

Carl made a minor point here about how we're using
``django.utils.unittest.TestCase``, which is essentially the ``TestCase`` class
from the Python ``unittest2` module and we're not using
``django.test.TestCase``, which adds a bunch of Django-specific stuff for
dealing with databases and transactions. The former is a little lighter, but it
shouldn't make a big difference, because the Django stuff is smart about not
doing database stuff if your test doesn't touch the database. There's `a nice
diagram in the Django docs showing the class relationships
<https://docs.djangoproject.com/en/dev/topics/testing/#testcase>`_.


Testing views
=============

(25:29 / 47:15 into the video)

*Unit* testing views is hard.
-----------------------------

* Views have many collaborators / dependencies.
* Templates, database, middleware, url routing...
* Write less view code!

A common problem seen in Django projects is **too much view code**. Views know
so much about your system, so it's tempting to put stuff in there because it's
easy. Carl doesn't like to see view functions longer than 10-12 lines.

Views are hard to write unit tests for, because they are where everything else
comes together and they are coupled to a lot of stuff.

One solution is to put less stuff in views since they're hard to test.

You might also consider testing views with functional tests rather than unit
tests.


If you unit test views
----------------------

(27:05 / 47:15 into the video)

* Use ``RequestFactory`` (`link in Django docs
  <https://docs.djangoproject.com/en/dev/topics/testing/#the-request-factory>`_).

  - An under-publicized but very useful class for generating fake
    ``HttpRequest`` objects to pass directly to view callables.

  - If you're using the `Django test client
    <https://docs.djangoproject.com/en/dev/topics/testing/?from=olddocs#module-django.test.client>`_,
    it is not a unit test. That goes through HTTP and thus is influenced by
    routing, middleware, etc.

* Call the view callable directly.

* Set up dependencies explicitly (e.g.: ``request.user``, ``request.session``)


``RequestFactory`` example
--------------------------

(28:08 / 47:15 into the video)

A hypothetical example of posting to a ``/locale/`` view to change the locale.

.. code-block:: python

    def test_change_locale(self):
        """POST sets 'locale' key in session."""
        request = RequestFactory().post("/locale/", {"locale": "es-mx"})
        request.session = {}

        change_locale(request)

        self.assertEqual(request.session["locale"], "es-mx")


Or don't.
---------

(29:14 / 47:15 into the video)

* Carl rarely unit tests views.
* Carl writes less view code, and covers it via functional tests.


Integration testing views
-------------------------

(29:39 / 47:15 into the video)

.. code-block:: python

    url = "/case/edit/{0}".format(case.pk)
    step = case.steps.get()
    response = self.client.post(url, {
        "product": case.product.id,
        "name": case.name,
        "description": case.description,
        "steps-TOTAL_FORMS": 2,
        "steps-INITIAL_FORMS": 1,
        "steps-MAX_NUM_FORMS": 3,
        "steps-0-step": step.step,
        "steps-0-expected": step.expected,
        "steps-1-step": "Click link.",
        "steps-1-expected": "Account active.",
        "status": case.status,
    })

* Django test client is sort of in a sour spot between a system test and a unit
  test.

  - It's a bad system test because it's easy to break it with a simple template
    change.

  - It's a bad unit test because it's going through way too much code.



WebTest!
--------

(31:16 / 47:15 into the video)

* `WebTest <http://webtest.pythonpaste.org/>`_ is a library from Ian Bicking.

* WebTest knows a lot less about Django, which is a good thing.

* WebTest interacts with your application through WSGI, which is much closer to
  how your users will interact with your application.

* WebTest knows more about HTML.

.. code-block:: python

    url = "/case/edit/{0}".format(case.pk)
    form = self.app.get(url).forms["case-form"]
    form["steps-1-step"] = "Click link."
    form["steps-1-expected"] = "Account active."

    response = form.submit()

* WebTest parses the form HTML and can submit it like a browser would.

* Notice how much simpler the WebTest example is compared to the Django test
  client example. WebTest eliminates a lot of boilerplate.


The markup matters.
-------------------

(32:51 / 47:15 into the video)

* If it can break, it should be tested.
* It can especially break forms.
* The output of your view is an HTTP response; the template + context is an
  implementation detail.


WebTest > django.test.Client
----------------------------

(34:12 / 47:15 into the video)

* System tests are easier and faster to write.

* Tests give you more confidence that the view works.

* There is also a project called `django-webtest
  <http://pypi.python.org/pypi/django-webtest>`_ that tries to integrate
  WebTest with Django.

.. code-block:: python

    self.assertEqual(response.json, ["one", "two", "three"])

    self.assertEqual(resp.html.find("a", title="Login").href, "/login/")

WebTest has some nice features:

* It parses JSON.

* It parses HTML (using `BeautifulSoup
  <http://www.crummy.com/software/BeautifulSoup/>`_ or `lxml
  <http://lxml.de/>`_).


In-browser testing
==================

(34:54 / 47:15 into the video)

More and more functionality depends on both JS and server. Needs to be tested
too.

Is easier than you think.
-------------------------

* Especially in Django 1.4.

* ``pip install selenium``

* ``LiveServerTestCase`` (`Django docs on LiveServerTestCase
  <https://docs.djangoproject.com/en/dev/topics/testing/#django.test.LiveServerTestCase>`_)

  - ``LiveServerTestCase`` runs the development server in a separate thread.


.. code-block:: python

    from django.test import LiveServerTestCase
    from selenium.webdriver.firefox.webdriver import WebDriver

    class MySeleniumTests(LiveServerTestCase):

        @classmethod
        def setUpClass(cls):
            cls.selenium = WebDriver()
            super(MySeleniumTests, cls).setUpClass()

        @classmethod
        def tearDownClass(cls):
            super(MySeleniumTests, cls).tearDownClass()
            cls.selenium.quit()

        def test_login(self):
            self.selenium.get("%s%s" % (self.live_server_url, "/login/"))
            username_input = self.selenium.find_element_by_name("username")
            username_input.send_keys("myuser")
            password_input = self.selenium.find_element_by_name("password")
            password_input.send_keys("secret")


What type of test to write?
===========================

(36:17 / 47:15 into the video)

Carl's rules of thumb:

* Write system tests for your views.
* Write Selenium tests for Ajax, other JS/server interactions.
* Write as few of the above 2 as possible.
* Write unit tests for everything else (not strict).
  - e.g.: when testing a ``ModelForm``, might not bother to mock out the model.
* Test each case (code branch) where it occurs.
* One assert/action per test case method.

  - One assert is stricter.

  - danger of multiple asserts is that if one assert fails, you don't know
    whether the other ones would pass or fail.

* Very rough guidelines; what works for Carl. Not strict; e.g. tests for a
  ``ModelForm`` don't mock the model.

* You should really avoid multiple step tests -- it's tempting but makes
  debugging tests harder.


Example of testing a view
-------------------------

(38:43 / 47:15 into the video)

.. code-block:: python

    def add_quote(request):
        if request.method == "POST":
            form = QuoteForm(request.POST)
            if form.is_valid():
                return redirect("quote_list")
        else:
            form = QuoteForm()

        return TemplateResponse(
            request,
            "add_quote.html",
            {"form": form},
            )

* This view should have 3 tests. Model/form special cases should be unit
  tested. And views shouldn't get much more complex.


Testing documentation
=====================

(40:00 / 47:15 into the video)

"We have always been at war with doctests"
------------------------------------------

* doctests let you put executable code examples in your docs and have the
  examples executed and verified (`Django docs on doctests
  <https://docs.djangoproject.com/en/dev/topics/testing/#writing-doctests>`_).

* Not entirely fair.

* Doctests are great.

* For testing documentation examples.


You have Sphinx docs.
---------------------

(40:59 / 47:15 into the video)

* Right?

You have API code examples.

In your Sphinx docs.

You can add stuff to make sure that examples in Sphinx docs are tested. In any
test file:

.. code-block:: python

    def load_tests(loader, tests, ignore):
        path = os.path.join(
            settings.BASE_PATH,
            "docs",
            "examples.rst",
            )

        tests.addTests(
            doctest.DocFileSuite(path))

        return tests

Your examples are tested!

* Please don't abuse this.

  - If you start treating these like tests for your code instead of
    documentation, then it will get complex and you will have bad
    documentation.

* Keep them documentation first.

``@override_settings(ALLOW_COMMENTS=True)``
-------------------------------------------

(42:07 / 47:15 into the video)

Testing with a specific settings value - the anti-pattern:

.. code-block:: python

    def test_comments_allowed(self):
      old_allow = settings.ALLOW_COMMENTS
      settings.ALLOW_COMMENTS = True
      try:
        # ...
      finally:
        settings.ALLOW_COMMENTS = old_allow

A better way to do this would be using `Michael Foord's Mock library`_.

In Django 1.4, using the ``@override_settings`` decorator (`Django docs on
override_settings
<https://docs.djangoproject.com/en/dev/topics/testing/#django.test.utils.override_settings>`_):

.. code-block:: python

    @override_settings(ALLOW_COMMENTS=True)
    def test_comments_allowed(self):
      # ...

Questions?
==========

(42:51 / 47:15 into the video)

Note that the following questions and answers are not verbatim quotes. I have
paraphrased and summarized.

Question: Any comments on the various Django nose modules that are around?

Carl: has used :py:mod:`django-nose` and is pretty happy with :ref:`unittest2
test discovery <unittest-test-discovery>`. Django's ``TEST_RUNNER`` setting
makes it pretty easy to swap in whatever test runner you like.

Question: Ever felt like you need to test the formset implementation?

Carl: If I were testing something in the formset, I would use a unit test for the formset class.

Question: More something that involves interaction between the formset and the view...or is this a code smell?

(They went back and forth a little bit and I didn't follow it too well).

Question: Thank you for your work on pip and virtualenv. I try not to repeat
myself and create reusable apps. Any advice on how to test reusable apps,
especially if I intend to put them on `PyPI <http://pypi.python.org/pypi>`_?

Carl: Don't make the mistake of putting the tests for your app in a
:file:`tests.py`, because then other people who use your app will be forced to
run your tests when they don't really need to. Better to put your tests in
another directory and then set up a test runner for your reusable app to run
them for you.
