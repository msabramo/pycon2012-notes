*********************************************************************************
web2py: ideas we stole and ideas we had by Massimo Di Pierro of DePaul University
*********************************************************************************

**Presenters**:  `Massimo Di Pierro <https://us.pycon.org/2012/speaker/profile/139/>`_ of DePaul University

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/112/

**Slides**: http://dl.dropbox.com/u/18065445/Slides/PySFTalkSlides.pdf

**Video**: https://www.youtube.com/watch?v=M5IPlMe83yI&list=PLBC82890EA0228306&index=16&feature=plpp_video

**Video running time**: 31:55


"Thanks Django, Rails, TG, Flask, CherryPy, Mako, web.py, ..."

Main features
=============

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
=====

Included APIs
=============

* generation and parsing: HTML / XML / RSS / JSON
* web services: JSON / JSON-RPC / XML / XML-RPC / AMF
* document generation WIKI, CSV, RTF, LATEX, PDF
* 10 different SQL dialects and Google App Engine
* Role based access control with login plugins local, OpenID, OAuth,
  1 and 2, Janrain, LDAP Consumer + Provider Central Authentication Service
* sending SMS, accepting Credit Card payments
* internationalization, cron jobs, multi-tenancy, ..


web2py architecture
===================

web2py modules
==============

Admin - Design
==============

web2py applications
===================

Architecture of applications
============================

Complete application ("friends")
================================

Controllers
===========

Routing (in, out, onerror)
==========================

Templates
=========

App-Admin
=========

Models
======

Queries
=======

Thread locals
=============

Multi-version/No-conflicts
==========================

Role based access control
=========================

Web services
============

.. code-block:: python

    @service.json
    @service.xml
    @service.jsonrpc
    @service.xmlrpc
    @service.soap
    @service.amfrpc3('domain')

Record versioning
=================

Modularity with digitally signed URLs
=====================================

Federated authentication
========================

Multi-tenancy
=============

GAE deployment
==============

Web translation
===============

Error logging
=============

Who uses web2py?
================

3000 registered users

Conclusions
===========

* web2py has been bround for since 2007
* 50% was rewritten in 2010 while mantaining backward compatibility
* Some like it, some find it useful
* Give it a try!

