********************************************************************************************************
IPython: Python at your fingertips by Fernando Pérez, Min Ragan-Kelley, Brian E. Granger, Thomas Kluyver
********************************************************************************************************

**Presenters**:  Fernando Pérez and others

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/121/

**Slides**:

**Video**: https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video

**Video running time**: 39:58

Resources:

* http://ipython.org/
* http://github.com/ipython
* https://github.com/ipython/ipython-in-depth

`(00:26)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=00m26s>`_
Talk in two parts:

1. Brief overview of IPython
2. Demo of the new IPython web notebook

Brief overview of IPython
=========================

Why IPython?
------------

`(00:40) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=00m40s>`_

the "I" in IPython is for "Interactive".

In scientific computing, we typically *don't know what we're doing*.

Exploratory computing is *not just for scientists*.


Python an excellent *base* for an interactive environment
---------------------------------------------------------

`(01:45) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=01m45s>`_

The default interactive environment is just a starting point.

.. code-block:: python

    $ python
    >>> ls
    ...
    NameError: name 'ls' is not defined
    >>> os?
    ...
    SyntaxError: invalid syntax
    >>> execfile('~/scratch/err.py')
    ...
    IOError: [Errno 2] No such file or directory '~/scratch/err.py'
    >>> execfile('/home/fperez/scratch/err.py')
    ...
    NameError: name 'foo33' is not defined


We can do better
----------------

`(02:37) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=02m37s>`_

.. code-block:: python

    $ ipython
    In [1]: ls ~/scratch/err*py
    /home/fperez/scratch/err25.py
    ...
    In [2]: os?
    (One question mark - Brings up a whole bunch of info about the os module)
    In[3]: os??
    (Two question marks - Brings up even more info, displaying the code if possible, syntax highlighted)
    ...
    In[13]: run ~/scratch/error
    ...
    (Traceback in color and shows the relevant lines of code)


Terminal IPython vs. QT console vs. Web notebook
------------------------------------------------

`(04:13) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=04m13s>`_

.. code-block:: bash

                            ┌──────────────────┐
                        ⇔   | Terminal console |
                            └──────────────────┘

    ┌────────────────┐      ┌──────────────────┐
    | IPython kernel |  ⇔   |    Qt Console    |
    └────────────────┘      └──────────────────┘

                            ┌──────────────────┐
                        ⇔   |   Web notebook   |
                            └──────────────────┘


`(05:32)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=05m32s>`_
Several clients now.

`(05:56)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=05m56s>`_
Qt console

* Feels like a terminal emulator
* But it's a Qt rich text widget that can display graphics, color, multi-line editing, etc.

`(06:41)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=06m41s>`_
Microsoft Visual Studio 2010 integrated console

See http://pytools.codeplex.com/

`(07:44)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=07m44s>`_
Browser-based notebook

* rich text
* code
* plots...

`(08:30)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=08m30s>`_
Interactive and high-level parallel APIs

The same abstractions and communications machinery that controls a single
interactive IPython instance can control multiple IPython instances.

Exposes an *IPython cluster* which consists of an *IPython controller* and one
or more *IPython engines*.

Enables parallel computing.


A brief history of IPython
--------------------------

`(10:20) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=10m20s>`_

* October/November 2001: "just a little afternoon hack"

  - $PYTHONSTARTUP: ipython-0.0.1.py (259 lines)
  - IPP (Interactive Python Prompt) by Janko Hauser (Oceanography)
  - LazyPython by Nathan Gray (CalTech)

* 2002: Drop John Hunter's Gnuplot patches: matplotlib
* 2004: Brian Granger, Min Ragan-Kelly: Parallel on Twisted
* 2005-2009: Mayavi, Wx support, refactoring; slow period.
* 2010: Discover 0MQ, Enthought support.

  - Move to Git/GitHub.
  - Build Qt console (Evan Patterson)
  - Rewrite parallel support with ZeroMQ.
  - Python 3 port (Thomas Kluyver)

* 2011: Web Notebook


Some quick stats
----------------

`(13:06) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=13m06s>`_


Support
-------

`(13:40) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=13m40s>`_


IPython in brief
----------------

`(14:06) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=14m06s>`_

* A better Python shell
* Embeddable kernel and powerful interactive clients

  - Terminal
  - Qt console
  - Web notebook

* Flexible parallel computing


Demo of the web notebook
========================

`(14:26) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=14m26s>`_


An example of showing Twitter word frequencies
----------------------------------------------

`(14:40) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=14m40s>`_


Tour of IPython notebook features
---------------------------------

`(19:32)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=19m32s>`_

`(19:43) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=19m43s>`_
pwd works

`(19:48) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=19m48s>`_
mixing Python and shell - you can interpolate Python variables into shell using a $


Plotting
--------

`(20:17) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=20m17s>`_
matplotlib plots.


Niceties
--------

You can paste code in with Python prompts and IPython will filter out the Python prompts

Highlight standard error

Detailed tracebacks

`(21:00) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=21m00s>`_
IPython fetches output asynchronously

`(21:15) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=21m15s>`_
Cause the kernel to segfault, but your document is saved so you can restart your kernel and get back to work


Markdown
--------

`(21:42) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=21m42s>`_
You can create `Markdown <http://daringfireball.net/projects/markdown/>`_ cells


Mathematics
-----------

`(22:00) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=22m00s>`_
Mathematics including `LaTeX <http://www.latex-project.org/>`_ and `MathJax <http://www.mathjax.org/>`_


Displaying complex output types, such as images
-----------------------------------------------

`(22:14)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=22m14s>`_
Can display arbitrary output types such as images and add your own
representations

**Displaying a local image**

.. image:: http://cl.ly/0T131G0D0Q2g2x073N2k/Screen%20shot%202012-03-29%20at%201.09.52%20PM.png

**Displaying a remote image**

.. image:: http://cl.ly/3l0s0M08123Y120z3Q0h/Screen%20shot%202012-03-29%20at%201.12.50%20PM.png


Embedding video
---------------

`(23:29)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=23m29s>`_
Video - can embed videos like YouTube videos


Embedding iframes
-----------------

`(24:50) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=24m50s>`_


Loading external code
---------------------

`(26:11) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=26m11s>`_

* `matplotlib gallery <http://matplotlib.sourceforge.net/gallery.html>`_ and other galleries

* Drag and drop a ``.py`` in the dashboard

* Use ``%loadpy`` with any local or remote url


SymPy
-----

`(27:55) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=27m55s>`_


IPython clusters
----------------

`(29:42) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=29m42s>`_


Conclusion
----------

`(35:20) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=35m20s>`_
Done with demo; we have a booth; we're having a sprint


Questions
=========

`(36:20) <https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=36m20s>`_

Q: How to collaborate on an IPython notebook?

A: Unfortunately it's fairly limited right now.
You can share notebook files on GitHub.
No real-time synchronization.
You can share a notebook by sending a person the URL to the notebook (and you can enable SSL and password protection).
One person has to save the notebook and the other person needs to reload it.

`(37:40)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=37m40s>`_
Q: How to integrate with `Sphinx <http://sphinx.pocoo.org/>`_?

A: In a gist, we have a very simple `reST
<http://docutils.sourceforge.net/rst.html>`_ exporter. Improving the exporter
should be easy so that you can create a Sphinx-compatible notebook. It's not
hard to do, but we're busy.

It looks like this might do the trick??? https://github.com/ipython/nbconvert

`(38:32)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=37m40s>`_
Q: How feasible is it to use IPython as a replacement shell?

A: Ask Mark Wiebe of `NumPy <http://numpy.scipy.org/>`_ who uses the Qt console

`(39:00)
<https://www.youtube.com/watch?v=26wgEsg9Mcc&list=PLBC82890EA0228306&index=9&feature=plpp_video#t=39m00s>`_
Q: Debugging in the interactive notebook?

A: If you type qtconconsole then you can connect to a running kernel. We also plan to expose it via the web.



