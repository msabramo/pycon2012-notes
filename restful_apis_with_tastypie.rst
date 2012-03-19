***************************************************************************
RESTful APIs with Tastypie by Daniel Lindsley
***************************************************************************
or
How I learned to stop worrying and love the JSON


**Presenter**: `Daniel Lindsley
<https://us.pycon.org/2012/speaker/profile/54/>`_ (http://www.toastdriven.com/)
(`@daniellindsley <https://twitter.com/#!/daniellindsley>`_)

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/61/

**Slides**: http://speakerdeck.com/u/daniellindsley/p/restful-apis-with-tastypie

**Video**: http://pyvideo.org/video/673/restful-apis-with-tastypie

**Video running time**: 34:07


About the speaker
=================

* **Daniel** Lindsley
* Consulting & OSS as **Toast Driven**
* Primary author of **Tastypie**


What is Tastypie?
=================

* A **REST** framework for **Django**
* Designed for **extension**
* Supports both **Model** & **non-Model** data
* http://tastypieapi.org/


Philosophy
----------

* Make good use of HTTP

  - Try to be "of the internet" & use the REST methods/status codes properly

* Graceful degradation

  - Try to keep backwards compatibility & give users a gradual upgrade path

* Flexible serialization

  - not everyone wants JSON

* Flexible EVERYTHING

  - Customizability is a core feature

* Data can round-trip

  - Anything you can GET, you should be able to POST/PUT

* Reasonable defaults

  - but easy to extend

* URIs everywhere!

  - Make HATEOAS a reality


HATEOAS
-------

* "Hypermedia As The Engine Of Application State"

* Basically the user **shouldn't have to know anything** in advance

* All about explore-ability

* **Deep linking**

* http://en.wikipedia.org/wiki/HATEOAS


Tastypie
--------

* Builds on top of Django & plays nicely with other apps

* Full ``GET/POST/PUT/DELETE/PATCH`` (``PATCH``? See :rfc:`5789`)

* **Any** data source (not just Models)

* Designed to be extended

* Supports a variety of serialization formats:

  - JSON
  - XML
  - YAML
  - bplist

* Easy to add more

* HATEOAS by default (you'll see soon)

* Lots of hooks for customization

* Well-tested -- about 80% coverage

* Decently documented


Installation
============

``pip install django-tastypie``

``INSTALLED_APPS += ['tastypie']``

``$ manage.py syncdb``

Done.


Let's add an API for :mod:`django.contrib.auth`
===============================================

Setting it up
-------------

Set up your app:

.. code-block:: bash

    # Assuming we're in your project directory...
    $ cd <myapp>   # Substitute your app_name here
    $ mkdir api
    $ touch api/__init__.py
    $ touch api/resources.py
    # Done!

User Resource:

.. code-block:: python

    from django.contrib.auth.models import User
    from tastypie.resources import ModelResource

    class UserResource(ModelResource):
        class Meta:
            queryset = User.objects.all()

URLconf:

.. code-block:: python

    # In your ROOT_URLCONF...
    from tastypie.api import Api
    from <myapp>.api.resources import UserResource
    v1_api = Api()
    v1_api.register(UserResource())

    urlpatterns = patterns('',
        (r'^api/', include(v1_api.urls),
        # Then the usual...
    )


Trying it out
-------------

========    =========================================
--------    -----------------------------------------
Curl:       http://localhost:8000/api/v1/
Browser:    http://localhost:8000/api/v1/?format=json
========    =========================================

* ``/api/v1/`` - A list of all available resources
* ``/api/v1/user/`` - A list of all users
* ``/api/v1/user/2/`` - A specific user
* ``/api/v1/user/schema/`` - A definition of what an individual user consists of
* ``/api/v1/user/multiple/1;4;5/`` - Get those three users as one request

All serialization formats available (provided `lxml <http://lxml.de/>`_,
`PyYAML <http://pyyaml.org/wiki/PyYAML>`_, and `biplist
<https://github.com/wooster/biplist>`_ are installed).

* ``curl -H "Accept: application/xml" http://localhost:8000/api/v1/user/``
* http://localhost:8000/api/v1/user/2/?format=yaml

Serialization format negotiated by either ``Accepts`` header or the ``"?format=json" GET`` param

Pagination by default

Everyone has full read-only ``GET`` access

What's not there? (Yet)

* Leaking sensitive information!

  - ``email/password/is_staff/is_superuser``

* Ability to filter

* Authentication/Authorization

* Caching (disabled by default)

* Throttling (disabled by default)


Excluding fields
----------------

.. code-block:: python

    from django.contrib.auth.models import User
    from tastypie.resources import ModelResource

    class UserResource(ModelResource):
        class Meta:
            queryset = User.objects.all()
            excludes = ['email', 'password', 'is_staff', 'is_superuser']


Add BASIC Auth
--------------

.. code-block:: python

    from django.contrib.auth.models import User
    from tastypie.authentication import BasicAuthentication
    from tastypie.resources import ModelResource

    class UserResource(ModelResource):
        class Meta:
            # What was there before...
            authentication = BasicAuthentication()


Add filtering
-------------

.. code-block:: python

    from django.contrib.auth.models import User
    from tastypie.authentication import BasicAuthentication
    from tastypie.resources import ModelResource, ALL

    class UserResource(ModelResource):
        class Meta:
            # What was there before...
            filtering = {
                'username': ALL,
                'date_joined': ['range', 'gt', 'gte', 'lt', 'lte'],
            }


* Using ``GET`` params, we can now filter out what we want.

* Examples:

  - ``curl http://localhost:8000/api/v1/user/?username__startswith=a``
  - ``curl http://localhost:8000/api/v1/user/?date_joined__gte=2011-12-01``


Add authorization
-----------------

.. code-block:: python

    from django.contrib.auth.models import User
    from tastypie.authentication import BasicAuthentication
    from tastypie.authorization import DjangoAuthorization
    from tastypie.resources import ModelResource

    class UserResource(ModelResource):
        class Meta:
            # What was there before...
            authorization = DjangoAuthorization()


Add caching
-----------

.. code-block:: python

    from django.contrib.auth.models import User
    from tastypie.authentication import BasicAuthentication
    from tastypie.authorization import DjangoAuthorization
    from tastypie.cache import SimpleCache
    from tastypie.resources import ModelResource

    class UserResource(ModelResource):
        class Meta:
            # What was there before...
            cache = SimpleCache()


Add throttling
--------------

.. code-block:: python

    from django.contrib.auth.models import User
    from tastypie.authentication import BasicAuthentication
    from tastypie.authorization import DjangoAuthorization
    from tastypie.cache import SimpleCache
    from tastypie.resources import ModelResource
    from tastypie.throttle import CacheDBThrottle

    class UserResource(ModelResource):
        class Meta:
            # What was there before...
            throttle = CacheDBThrottle()


What's there now?
-----------------

* Everything we had before
* **Full** ``GET/POST/PUT/DELETE/PATCH`` access
* Only **registered users** can use the API & only perform actions on objects they're allowed to
* Object-level caching (``GET`` detail)
* Logged throttling that limits users to 150 reqs per hour
* The ability to filter the content


Extensibility
=============

Designed for extensibility
--------------------------

* Why classes?

  - Not because I'm OO-crazy.

  - It makes extending behavior trivial.

* Why so many classes?

  - Composition > Inheritance

* Why so many methods?

  - Hooks, hooks, hooks.

  - Also makes delegating to composition behaviors easy.

* Tastypie tries to use **reasonable defaults**:

  - You probably want **JSON**

  - You probably want **full** ``POST/PUT/DELETE`` by default

  - You probably want to use the Model's **default manager** unfiltered

* But Tastypie lets you change all these things

* Plug in custom classes/instances for things like:

  - Serialization
  - Authentication
  - Authorization
  - Pagination
  - Caching
  - Throttling

* ``Resource`` has lots of methods, many of which are pretty granular

* Override or extend as meets your needs

Customize serialization
-----------------------

* As an example, let's customize serialization

* Supports JSON, XML, YAML, bplist by default

* Let's disable everything but JSON and XML, then add a custom type for HTML

* To limit to just JSON and XML:

.. code-block:: python

    from django.contrib.auth.models import User
    from tastypie.resources import ModelResource
    from tastypie.serialization import Serializer

    class UserResource(ModelResource):
        class Meta:
            queryset = User.objects.all()
            excludes = ['email', 'password', 'is_staff', 'is_superuser']
            serializer = Serializer(formats=['json', 'xml'])


HTML serialization
------------------

.. code-block:: python

    from django.shortcuts import render_to_response
    from tastypie.serialization import Serializer
    import cgi
    from stringio import StringIO

    class TemplateSerializer(Serializer):
        formats = Serializer.formats + ['html']

        def to_html(self, data):
            template_name = 'api/api_detail.html'

            if 'objects' in data:
                template_name = 'api/api_list.html'

            return render_to_response(template_name, data)

        def from_html(self, content):
            form = cgi.FieldStorage(fp=StringIO(content))
            data = {}
            for key in form:
                data[key] = form[key].value
            return data

Using it:

.. code-block:: python

    from django.contrib.auth.models import User
    from tastypie.resources import ModelResource
    from myapp.api.serializers import TemplateSerializer

    class UserResource(ModelResource):
        class Meta:
            queryset = User.objects.all()
            excludes = ['email', 'password', 'is_staff', 'is_superuser']
            serializer = TemplateSerializer(formats=['json', 'xml', 'html'])


Fields
------

* Just like ``ModelForm``, you can control all of the exposed fields on a ``Resource/ModelResource``.

* Just like Django, you use a declarative syntax.

.. code-block:: python

    from django.contrib.auth.models import User
    from tastypie import fields
    from tastypie.resources import ModelResource

    class UserResource(ModelResource):
        # Provided they take no args, even callables work!
        full_name = fields.CharField('get_full_name', blank=True)

        class Meta:
            queryset = User.objects.all()
            excludes = ['email', 'password', 'is_staff', 'is_superuser']

* You can control how data gets prepared for presentation (``dehydrate``) or accepted from the user (``hydrate``).

* Happens automatically on fields with ``attribute=...`` set

.. code-block:: python

    def dehydrate_full_name(self, bundle):
        return bundle.obj.get_full_name()

    def hydrate_full_name(self, bundle):
        ...
        return bundle

Caching
-------

* The ``SimpleCache`` combined with ``Resource.cached_obj_get`` caches  **SINGLE** objects only!

* Doesn't cache the **serialized output**

* Doesn't cache the **list view**

Why?

* More complex behaviors get **opinionated** fast

* Tastypie would rather **be general** & give you the **tools** to build what you need

* Filters and serialization formats make it complex

* Besides...

What you actually want is **Varnish**
-------------------------------------

* https://www.varnish-cache.org/

* **Super-fast** caching reverse proxy in C

* Already caches by URI/headers

* Way **faster** than the Django request/response cycle

* ``POST/PUT/DELETE`` just pass through

* So put Varnish in front of your API (and perhaps the rest of your site) and **win** in the general case.

* Additionally, use Tastypie's internal caching to further speed up Varnish cache-misses.

* Easy to extend ``Resource`` to add in more caching

* If you get to that point, you're already serving way more load than I ever have.


Data source: not just models!
-----------------------------

* ``ModelResource`` is just a relatively thin (~300 lines) wrapper on top of ``Resource`` (~1200 lines)

* Just the ORM/Model bits.

* **So virtually everything in Tastypie is available to non-ORM setups.**

* By subclassing from ``Resource`` and overriding 3 to 9 methods, you can hook up any data source

* http://django-tastypie.readthedocs.org/en/latest/non_orm_data_sources.html


Example: A Solr-based resource (``GET``-only)
---------------------------------------------

(A fair amount of code)

* Takes some work but does a lot for you

* Docs have a more complete `example based on Riak
  <http://django-tastypie.readthedocs.org/en/latest/non_orm_data_sources.html#using-riak-for-messageresource>`_

* See also `django-tastypie-nonrel
  <https://github.com/andresdouglas/django-tastypie-nonrel>`_.


Piecrust: The extraction that failed
====================================

* Late 2011, tried extracting Tastypie to work anywhere (not just Django). It was called Piecrust.

* http://github.com/toastdriven/piecrust

* Close to functional but failed in terms of **complexity** and lack of **standardization**



