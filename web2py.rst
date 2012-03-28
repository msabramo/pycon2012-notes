*********************************************************************************
web2py: ideas we stole and ideas we had by Massimo Di Pierro of DePaul University
*********************************************************************************

**Presenters**:  `Massimo Di Pierro <https://us.pycon.org/2012/speaker/profile/139/>`_ of DePaul University

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/112/

**Slides**: http://dl.dropbox.com/u/18065445/Slides/PySFTalkSlides.pdf

**Video**: https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video

**Video running time**: 31:55


web2py - http://web2py.com/


Ideas we borrowed
=================

`(01:16) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=01m16s>`_

* Model View Controller on WSGI (like everyone else)
* w2p files (like Java's Web application ARchives)
* Optional routing (like Rails)
* Routing mechanism (like Django)
* Pure Python templating language (like Mako)
* Helpers (like Rails) but easier to remember: ``span``, ``a``, etc.
* ``request['xxx'] == request.xxx`` (like web.py)
* web based database interface (like Django admin)


Ideas we had
============

(`03:10) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=03m10s>`_

* A tool for both technical and non-technical people
* Always backwards compatible (since 2007, 2.5, 2.6, 2.7, pypy, jython)
* One click deploy (Windows and Mac binaries, USB drive)
* No configuration, no dependencies, and secure by default
* Everything has default (DRY)
* Multi project / multi db / share nothing by default
* Web based IDE (development, editor, deployment, management, translations, testing, debugger, version control) shell optional
* Automatic db migrations (CREATE and ALTER table)
* Plugins / Components / Ajax with digitally signed URLs

`(05:10) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=05m10s>`_

* Role based access control
* Every app is a Central Authentication Service provider (and optionally consumer)
* Built-in portable cron and master/workers task scheduler
* multi tenant apps with common fields & common filters
* record versioning (without breaking refs)
* embeddable CRUD and grid controls
* Package lots of 3rd party modules: pyfpdf, database drivers, credit card payment APIs, ...


Getting web2py
==============

`(06:32) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=06m32s>`_

.. code-block:: bash

    wget http://www.web2py.com/examples/static/web2py_src.zip
    unzip web2py_src.zip
    cd web2py
    web2py.py -a 'apassword' &

Gluon is 1.3 MB; smaller than most microframeworks


Multi-project
=============

`(09:24) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=09m24s>`_


Web based IDE
=============

`(09:43) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=09m43s>`_


Mobile IDE
==========

`(09:59) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=09m59s>`_


Web translation
===============

`(10:07) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=10m07s>`_


Tickets
=======

`(10:17) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=10m17s>`_


Model-View-Controller
=====================

`(10:40) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=10m40s>`_


Let's build something
=====================

`(11:08) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=11m08s>`_

* URL shortening service
* Bookmark service with tagging
* Click counter
* URL rating (WOT service)


Demo
====

Download and start web2py
-------------------------

`(11:44) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=11m44s>`_

.. code-block:: bash

    # download and start web2py
    wget http://www.web2py.com/examples/static/web2py_src.zip
    unzip web2py_src.zip
    cd web2py
    python2.7 web2py.py -a hello -p 8000 &
    open http://127.0.0.1:8000/welcome/

    # Shows the welcome page

    cd applications
    ls
    __init__.py
    __init__.pyc
    admin
    examples
    friends
    links
    welcome


Explore the admin interface
---------------------------

`(12:20) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=12m20s>`_

.. code-block:: bash

    open http://127.0.0.1:8000/admin/

    # Shows the administrative interface
    # Can edit files
    # Can get a web-based Python shell
    # Can run web-based tests
    # Can configure crontab
    # Can access Mercurial versioning
    # Can access error logs
    # Can upgrade web2py
    # Can use new application wizard


Create a new application from the shell
---------------------------------------

`(13:38) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=13m38s>`_

.. code-block:: bash

    mkdir myapp
    cp -r welcome/* myapp/
    cd myapp
    open http://127.0.0.1:8000/myapp/default/index

    rm controllers/default.py
    edit controllers/default.py

.. code-block:: python

    def index():
        return "hello world"

    def oops():
        1/0

.. code-block:: bash

    open http://127.0.0.1:8000/myapp/default/index

http://127.0.0.1:8000/myapp/default/index displays "hello world"


Show error with ticket and traceback
------------------------------------

`(14:25)
<https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=14m25s>`_
http://127.0.0.1:8000/myapp/default/oops displays "Internal error" with a link
to a ticket with the traceback.


Edit a model
------------

`(14:37) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=14m37s>`_

.. code-block:: bash

    edit models/mydb.py

.. code-block:: python

    Link = db.define_table(
        'link',
        Field('url', unique=True))

.. code-block:: bash

    open http://127.0.0.1:8000/myapp/appadmin


App administrative interface
----------------------------

`(14:48) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=14m48s>`_


Add more fields to our model
----------------------------

`(15:00) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=15m00s>`_

.. code-block:: bash

    edit models/mydb.py

.. code-block:: python

    Link = db.define_table(
        'link',
        Field('url', unique=True),
        Field('visits', 'integer', default=0),
        Field('screenshot', 'upload', writable=False),
        format = '%(url)s')

    Bookmark = db.define_table(
        'bookmark',
        Field('Link', 'reference link', writable=False),
        Field('category', requires=IS_IN_SET(['work', 'personal'])),
        Field('tags', 'list:string'),
        auth.signature)

    def toCode(id):
        s, c = 'GKys67LJPDAFvcEp9rkw8...', ''
        while id: c,id = c+s[id % 55], id//55
        return c

    def toInt(code):
        s, id = 'GKys67LJPDAFvcEp9rkw8...', 0
        for i in range(len(code)): id += s.find(code[i])*55**i
        return id

    def shorten(id, row):
        s = URL('visit', args=toCode(id), scheme=True)
        return A(s, _href=s)

    Bookmark.link.represent = shorten
    Bookmark.id.readable = False
    Bookmark.is_active.readable = False
    Bookmark.is_active.writable = False

.. code-block:: bash

    open http://127.0.0.1:8000/myapp/appadmin

`(17:25)
<https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=17m25s>`_
Note that by adding a field to a model, web2py automatically detects it and
does the SQL ALTER TABLE stuff.


Integrating with web services
-----------------------------

`(17:55) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=17m55s>`_

* Integrate with thumbalizer and WOT (Web Of Trust)


Edit the menu
-------------

`(18:53) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=18m53s>`_

.. code-block:: bash

    edit models/menu.py

.. code-block:: python

    response.menu = (
        (T('Home'), False, URL('default', 'index')),
        (T('My Bookmarks'), False, URL('default', 'bookmarks')),
        ]


Edit the controller
-------------------

`(19:18) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=19m18s>`_


Show how it works
-----------------

`(21:40) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=21m40s>`_


Customize routes
----------------

`(22:54) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=22m54s>`_

Edit ``routes_in`` and ``routes_out`` in :file:`routes.py`


Authentication using Janrain
----------------------------

`(23:49) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=23m49s>`_

Janrain auth - ldap, oauth1, oauth2, google, openid, dropbox

All you have to do is copy :file:`janrain.key` into :file:`private` folder.


Edit templates
--------------

`(25:15) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=25m15s>`_


Create web services
-------------------

`(28:25) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=28m25s>`_

The old way

.. code-block:: python

    @service.json
    @service.xml
    @service.jsonrpc
    @service.xmlrpc
    @service.amfrpc3('domain')
    @service.soap
    def linksby(key=''):
        return db(Link.id==Bookmark.link)\
            (Bookmark.tags.contains(key)).select(Link.ALL, distinct=True)

The new way

.. code-block:: python

    @request.restful()
    def api():
        def POST():
            raise HTTP(501)
        def GET(keys=''):
            return dict(result=linksby(key).as_list())
        return locals()

    def call(): return service()


The scheduler
-------------

`(30:08) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=30m08s>`_


Deploy remotely
---------------

`(31:19) <https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video#t=31m19s>`_

Package the app into a w2p file.



Notes from PDF slides
=====================

These are notes from the slides at
http://dl.dropbox.com/u/18065445/Slides/PySFTalkSlides.pdf, which did not map
closely to the talk.

Main features
-------------

* One Instance - Many Applications (hot plug and play)
* Web based Integrated Development Environment
* Web based Database administration (for each app)
* Each application can connect to multiple Databases
* Writes SQL for you
* Strong on Security (no SQL Injections, XSS, CSRF, ..., audited)
* Built-in ticketing system (logs all errors)
* Runs everywhere (it is written in Python)
* Requires NO Installation (just download and unzip)
* Has no coniguration files and no third party dependencies
* Can run off a USB drive
* Always backward compatible (since 2007 and on...)
* 50+ of developers already involved


Admin
-----

Included APIs
-------------

* generation and parsing: HTML / XML / RSS / JSON
* web services: JSON / JSON-RPC / XML / XML-RPC / AMF
* document generation WIKI, CSV, RTF, LATEX, PDF
* 10 different SQL dialects and Google App Engine
* Role based access control with login plugins local, OpenID, OAuth,
  1 and 2, Janrain, LDAP Consumer + Provider Central Authentication Service
* sending SMS, accepting Credit Card payments
* internationalization, cron jobs, multi-tenancy, ..


web2py architecture
-------------------

web2py modules
--------------

Admin - Design
--------------

web2py applications
-------------------

Architecture of applications
----------------------------

Complete application ("friends")
--------------------------------

Controllers
-----------

Routing (in, out, onerror)
--------------------------

Templates
---------

App-Admin
---------

Models
------

Queries
-------

Thread locals
-------------

Multi-version/No-conflicts
--------------------------

Role based access control
-------------------------

Web services
------------

.. code-block:: python

    @service.json
    @service.xml
    @service.jsonrpc
    @service.xmlrpc
    @service.soap
    @service.amfrpc3('domain')

Record versioning
-----------------

Modularity with digitally signed URLs
-------------------------------------

Federated authentication
------------------------

Multi-tenancy
-------------

GAE deployment
--------------

Web translation
---------------

Error logging
-------------

Who uses web2py?
----------------

3000 registered users

Conclusions
-----------

* web2py has been bround for since 2007
* 50% was rewritten in 2010 while mantaining backward compatibility
* Some like it, some find it useful
* Give it a try!

