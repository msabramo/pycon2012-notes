***************************************************************************
Django Templating: More Than Just Blocks
***************************************************************************

**Presenter**: `Christine Cheung
<https://us.pycon.org/2012/speaker/profile/70/>`_ (http://www.xtine.net/)
(`@plaidxtine <https://twitter.com/#!/plaidxtine>`_)

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/288/

**Slides**: http://speakerdeck.com/u/xtine/p/django-templating-more-than-just-blocks

**Video**: http://pyvideo.org/video/697/django-templating-more-than-just-blocks


Outline

* Intro to Templating
* Effective Use of Built-In Tags
* Extending Templates
* Template Loading
* What's New


Intro to Templating
===================

Django Templating 101
---------------------

* This is the End User Experience
* Balance between power and ease
* Design


Helpful tools
-------------

* Django template theme for your IDE

  - syntax highlighting, autocompletion

* `django-debug-toolbar <https://github.com/django-debug-toolbar/django-debug-toolbar>`_

  - template paths, session variables

* Print out tag/filter reference guide

  * http://media.revsys.com/images/django-1.4-cheatsheet.pdf
  * https://docs.djangoproject.com/en/dev/ref/templates/builtins/


Folder and file structure
-------------------------

* Keep templates in one place


Style guide
-----------

* Think :pep:`8` coding conventions

  - consistent spacing

* ``{% load %}`` all template tags up top

* try ``{# comment #}`` rather than ``<!-- this -->``

  - also, ``{% comment %}{% endcomment %}``


Effective Use of Built-In Tags
==============================

The Basics
----------

* Start with :file:`base.html`

    .. code-block:: html

        <!doctype html>
        <head>
            <title>{% block title %}demo{% endblock title %}</title>
        </head>
        <body>

        {% block content %}{% endblock content %}

        </body>
        </html>

* Then have pages inherit from it

    .. code-block:: html

        {% extends "base.html" %}

        {% block title %}the foo page{% endblock title %}

        {% block content %}
            <div id="foo">
                this is a bar.
            </div>
        {% endblock content %}


Common blocks
-------------

I use these in practically every project:

* title
* meta_tags, robots,
* extra_head (CSS, etc.),
* content
* extra_js (so that JavaScript can go at bottom of page which improves page-load time)


Block practices
---------------

* End your block structures

  - ``{% block title %}foo{% endblock title %}``
  - instead of ``{%block title %}foo{% endblock %}``

* Can't repeat blocks

  - however: `context processor
    <https://docs.djangoproject.com/en/dev/ref/templates/api/#subclassing-context-requestcontext>`_,
    `include
    <https://docs.djangoproject.com/en/dev/ref/templates/builtins/?from=olddocs#include>`_,
    `custom template tag
    <https://docs.djangoproject.com/en/dev/howto/custom-template-tags/>`_

* Don't "over block"


Including templates
-------------------

* ``{% include "snippet.html" %}``

  - great for repeating template segments

* try not to include in an include -- gets confusing


Variables
---------

* Tend to be objects passed from a view

  - *Modify* objects with **filters**

    * ``{{ variable | lower }}``

  - *Loop* through etc. using **tags**

    * ``{% if variable %}foo{% else %}bar{% endif %}``
    * ``{% for entry in blog_entries %}<h2>{{ entry.title }}</h2><p>{{ entry.body }}</p>{% endfor %}``

  - You can also create your own filters and tags (see `Django docs on custom
    template tags and filters
    <https://docs.djangoproject.com/en/dev/howto/custom-template-tags/>`_)


Security
--------

By default, Django's security is rather solid on the template side of things...

* but if you use **safe** or ``{% autoescape %}``

  - *** make sure you sanitize the data! ***


URLs
----

Name ``{% url %}`` tags as much as possible

* define `URL patterns
  <https://docs.djangoproject.com/en/dev/topics/http/urls/>`_ in
  :file:`urls.py`

  - ``url(r'^foo/$', foo, name="foo"),``
  - ``<a href="{% url "foo" %}">foo</a>``

``{{ STATIC_URL }}css/style.css``

  - Not ``/static/css/style.css``


Forms
-----

For heavy form action, take a look at:

- `django-floppyforms <http://django-floppyforms.readthedocs.org/>`_ (HTML 5)
- `django-crispy-forms <http://django-crispy-forms.readthedocs.org/>`_ (used to
  be `django-uni-form <https://github.com/pydanny/django-uni-form>`_)

.. code-block:: django

    {% include form.html %}

- ``as_ul`` (`docs
  <https://docs.djangoproject.com/en/dev/ref/forms/api/#as-ul>`_) makes more
  sense than ``as_p`` or ``as_table``


More than one way
-----------------

There are multiple ways to accomplish the same task.

No ultimately right or wrong way

* use what suits you or your team

An example

The long way:

.. code-block:: django

    {% if foo.bar %}
        {{ foo.bar }}
    {% else %}
        {{ foo.baz }}
    {% endif %}

or the shorter way:

.. code-block:: django

    {% firstof foo.bar foo.baz %}

Extending templates
===================

Custom Tags and Filters
-----------------------

.. raw:: html

    <pre>
    demo/
        models.py
        templatetags/
            __init__.py
            <b>demo_utils.py</b>
        view.py
    </pre>

Given that we have template tags in :file:`demo/templatetags/demo_utils.py`

.. code-block:: django

    {% load demo_utils %}


Making a Custom Filter
----------------------

.. code-block:: python

    from django import template
    register = template.Library()

    @register.filter(name='remove')

    def cut(value, argument):
        # remove passed arguments from value
        return value.replace(argument, '')

.. code-block:: django

    {{ foo|remove:'bar' }}

.. code-block:: python

    @register.filter
    def lower(value):
        # lowercased value with no passed arguments
        return value.lower()

.. code-block:: django

    {{ foo|lower }}


Making a Custom Tag
-------------------

Tags are a bit more complex

* two steps: compiling and rendering

Decide its purpose

* but start simple

better to have many tags that do many things rather than one tag that does many things


A Simple Example
----------------

.. code-block:: django

    <p>
        It is now
        {% current_time "%Y-%m-%d %I:%M %p" %}
    </p>


Simple Tag
----------

.. code-block:: python

    from django import template

    register = template.Library()

    @register.simple_tag
    def current_time(format_string):
        try:
            return datetime.datetime.now().strftime(str(format_string))
        except UnicodeEncodeError:
            return 'oh noes current time borked'


Nodes and Stuff
---------------

.. code-block:: python

    import datetime
    from django import template
    register = template.Library()

    @register.tag(name="current_time")

    def do_current_time(parser, token):
        try:
            tag_name, format_string = token.split_contents()
        except ValueError:
            msg = '%r tag requires a single argument' % token.split_contents()[0]
            raise template.TemplateSyntaxError(msg)


    class CurrentTimeNode(template.Node):
        def __init__(self, format_string):
            self.format_string = str(format_string)

        def render(self, context):
            now = datetime.datetime.now()
            return now.strftime(self.format_string)


Easier Template Tag Creation
----------------------------

`django-templatetag-sugar <https://github.com/alex/django-templatetag-sugar>`_

* makes it simple to define syntax for a tag

`django-classy-tags <https://github.com/ojii/django-classy-tags>`_

* class-based template tags
* extensible argument parse for less boilerplate


DO NOT!!!
---------

Do not write a template tag that runs logic or at worst, even run Python from a
custom tag

* it defeats purpose of a templating language
* dangerous
* difficult to support


Loading templates
=================

Template loading logic
----------------------

Use cases

``TEMPLATE_LOADERS`` setting (`Django docs <https://docs.djangoproject.com/en/dev/ref/settings/#template-loaders>`_)

.. code-block:: python

    from django.conf import settings
    from django.template import TemplateDoesNotExist

    def load_template_source(template_name, template_dirs=None):
        for filepath in get_template_sources(template_name, template_dirs):
            try:
                # load in some templates yo
            except IOError:
                pass

        raise TemplateDoesNotExist(template_name)


Replacing the templating engine
-------------------------------

You can replace the built in templating engine

* `Jinja2 <http://jinja.pocoo.org/docs/>`_, `Mako
  <http://www.makotemplates.org/>`_, `Cheetah
  <http://www.cheetahtemplate.org/>`_, etc. (Jinja probably the most popular)

But why?

* More familiar with another templating language
* Performance boost
* Different logic control and handling

but you risk "frankensteining" your project.


Jinja2 and Django and You
-------------------------

Pros

* functions callable from templates - don't have to write tags and filters
* loop controls - more powerful flow control
* multiple filter arguments
* slight performance increase

Cons

* more dependencies and overhead
* extra time spent on development and support
* risk putting too much logic in templates
* minimal speed increase


Speeding Up Templates
---------------------

Cache template loader


`django-template-preprocessor <https://github.com/citylive/django-template-preprocessor>`_

- compiles template files

`django-pancake <https://github.com/adrianholovaty/django-pancake>`_ 

- from `Adrian Holovaty <http://www.holovaty.com/>`_ (one of the creators of Django)
- flattens template files

but also remember other bottlenecks...


New in Django 1.4
-----------------

`Django 1.4 release notes <https://docs.djangoproject.com/en/dev/releases/1.4/>`_

Custom project and app templates

* startapp/startproject -- template (`docs on Django 1.4 custom project and app
  templates
  <https://docs.djangoproject.com/en/dev/releases/1.4/#custom-project-and-app-templates>`_)
* combine with your favorite boilerplate

Else if

* ``{% elif %}`` (`docs on minor features <https://docs.djangoproject.com/en/dev/releases/1.4/#minor-features>`_)


Questions
=========

Questions????

