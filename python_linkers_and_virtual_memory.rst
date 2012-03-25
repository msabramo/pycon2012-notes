***************************************************************************
Python, Linkers, and Virtual Memory by Brandon Rhodes
***************************************************************************

**Presenter**: `Brandon Craig Rhodes
<https://us.pycon.org/2012/speaker/profile/310/>`_ (http://rhodesmill.org/brandon/)
(`@brandon_rhodes <http://twitter.com/#!/brandon_rhodes>`_)

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/455/

**Slides**: http://rhodesmill.org/brandon/talks/2012-03-pycon/linkers-memory/

**Video**: http://pyvideo.org/video/717/python-linkers-and-virtual-memory

**Video running time**: 31:16


Imagine a computer with 1,000 bytes of addressable RAM...
=========================================================

(00:19 / 36:22 into the video)

* numbered 0 through 999, each storing a byte

==================    ===============
**Memory address**    **Byte stored**
------------------    ---------------
               999                128
               998                255
               997                254
               996                128
               ...                ...
                 3                  2
                 2                  0
                 1                104
                 0                 77
==================    ===============


Pages
-----

We split memory into **pages** by grouping addresses that share a common prefix

(00:29 / 36:22 into the video)

If we defined our

**page size = 100 bytes**

Then our **1,000** bytes of main memory = **10 pages**

and **page 3** would include all addresses in the **3xx** range.

.. code-block:: python

       •
      400
    ┌─399─┐
    │ 398 │
    │ 397 │
    │  •  │
    │  •  │   Page 3
    │  •  │
    │ 302 │
    │ 301 │
    └─300─┘
      299
       •


Page tables
-----------

(00:49 / 36:22 into the video)

What is a page table?

.. code-block:: python

       ┌─────┐
       │ CPU │
       └─────┘
          ↕
    ┌────────────┐
    │ Page Table │
    └────────────┘
          ↕
       ┌─────┐
       │ RAM │
       └─────┘

It sits in between your CPU and the RAM on your machine and

**rewrites** the leading digits of every processor memory access **before** the
RAM chip sees the request.

.. code-block:: python

    Virtual Physical
    ┌──────────────┐
    │ 9--  →  59-- │
    │ 8--  →  63-- │
    │ 7--  →     - │
    │ 6--  →  61-- │
    │ 5--  →  60-- │
    │ 4--  →     - │
    │ 3--  →     - │
    │ 2--  →     - │
    │ 1--  →  36-- │
    │ 0--  →  35-- │
    └──────────────┘

The addresses that your processor asks for are called **virtual**.

The actual addresses delivered to RAM are called **physical**.

* "Read virtual byte **821**" => "Read physical byte **6321**"
* "Read virtual byte **190**" => "Read physical byte **3690**"
* "Write virtual byte **522**" => "Write physical byte **6022**"
* "Read virtual byte **700**" => **Page fault**

.. seealso::

    http://en.wikipedia.org/wiki/Page_table


Page Faults
-----------

(01:46 / 36:22 into the video)

Your program is paused and the OS regains control.

We will see that the OS has several options about how to respond.

.. seealso::

    http://en.wikipedia.org/wiki/Page_fault


Q: What is stored in the bytes of memory pages?
-----------------------------------------------

(01:55 / 36:22 into the video)

A: Everything your program needs

* Executable code
* Stack - variables
* Heap - data structures

(02:10) These resources tend to **load gradually** as your program runs.

(02:16) Imagine running your editor...

Which I will not name to avoid obvious religious wars... :-)

You say: "**Run my editor!**"

So the OS loads it into memory along with any libraries it requires.

.. code-block:: python

    ┌──────────────┐
    │ 9--  →     - │
    │ 8--  →     - │
    │ 7--  →     - │
    │ 6--  →     - │
    │ 5--  →     - │
    │ 4--  →     - │
    │ 3--  →     - │
    │ 2--  →     - │
    │ 1--  →  36-- │ libcurses
    │ 0--  →  35-- │ editor “binary” (program)
    └──────────────┘

(02:32) The editor's :func:`main()` function starts calling other functions.

(02:36) So the OS automatically starts allocating new stack pages as that call
graph grows.

The stack usually starts at a high address and grows downward.

.. code-block:: python

    ┌──────────────┐
    │ 9--  →  59-- │ stack (“bottom” = oldest)
    │ 8--  →  63-- │ stack (“top” = newest)
    │ 7--  →     - │
    │ 6--  →     - │
    │ 5--  →     - │
    │ 4--  →     - │
    │ 3--  →     - │
    │ 2--  →     - │
    │ 1--  →  36-- │ libcurses
    │ 0--  →  35-- │ editor “binary” (program)
    └──────────────┘

Notice the fairly random physical addresses that can come from anywhere in RAM,
because the association is a free one.

(02:57) Then the editor needs space for data structures like lists and
dictionaries.

These go in another growing memory area called the **heap**.

Which instead of holding variables that go away when your function returns
(i.e.: the stack), holds persistent data structures.

.. code-block:: python

    ┌──────────────┐
    │ 9--  →  59-- │ stack (“bottom” = oldest)
    │ 8--  →  63-- │ stack (“top” = newest)
    │ 7--  →     - │
    │ 6--  →  61-- │ heap (file being edited)
    │ 5--  →  60-- │ heap (settings)
    │ 4--  →     - │
    │ 3--  →     - │
    │ 2--  →     - │
    │ 1--  →  36-- │ libcurses
    │ 0--  →  35-- │ editor “binary” (program)
    └──────────────┘


Page Table benefits
-------------------

(03:22 / 36:22 into the video)

What are the benefits of this level of indirection?

(that has to be implemented in hardware and invoked every time your process hits main memory)

**Security**

* Processes can't see each other's pages
* Processes can't see OS pages
* The OS wipes reallocated pages with 0s.

**Stability**

* Processes can't overwrite each other.
* Processes can't crash the OS.
* You can't corrupt another process's memory pages because they're not even
  visible; they don't exist to you if they're not in the page table.


Protection
----------

(04:22 / 36:22 into the video)

Real page tables include **read** and **write** bits.

In general you can **write** your own stack and heap, but only **read**
executables and libraries.

.. code-block:: python

    ┌────────────────┐
    │ 9--  →  59-- w │ stack
    │ 8--  →  63-- w │ stack
    │ 7--  →       - │
    │ 6--  →  61-- w │ heap
    │ 5--  →  60-- w │ heap
    │ 4--  →       - │
    │ 3--  →       - │
    │ 2--  →       - │
    │ 1--  →  36-- r │ libcurses
    │ 0--  →  35-- r │ binary
    └────────────────┘

* Write bits set on the stack and heap
* Only the read bit set on the editor binary and the library.


Segmentation Faults
-------------------

(04:54 / 36:22 into the video)

For an illegal read or write...

Then instead of responding to your page fault with more memory, the OS will
stop you dead with a **Segmentation Fault**.

Which basically means a page fault that made the OS angry. :-)

(05:11 / 36:22 into the video)

"Read from **331**"

.. code-block:: python

      ┌────────────────┐
      │ 9--  →  59-- w │
      │ 8--  →  63-- w │
      │ 7--  →       - │
      │ 6--  →  61-- w │
      │ 5--  →  60-- w │
      │ 4--  →       - │
    → │ 3--  →       - │  → SEGMENTATION FAULT
      │ 2--  →       - │
      │ 1--  →  36-- r │
      │ 0--  →  35-- r │
      └────────────────┘

causes a **segmentation fault** because there's no page there.

"Write to **101**"

.. code-block:: python

      ┌────────────────┐
      │ 9--  →  59-- w │
      │ 8--  →  63-- w │
      │ 7--  →       - │
      │ 6--  →  61-- w │
      │ 5--  →  60-- w │
      │ 4--  →       - │
      │ 3--  →       - │
      │ 2--  →       - │
    → │ 1--  →  36-- r │ → SEGMENTATION FAULT
      │ 0--  →  35-- r │
      └────────────────┘

causes a **segmentation fault** because we're trying to write to a read-only page.

.. seealso::

    http://en.wikipedia.org/wiki/Segmentation_fault


Sharing
-------

(05:28 / 36:22 into the video)

Read-only pages give the OS a new superpower: **Sharing**!

Read-only binaries and libraries can be safely and securely shared among many
processes!

The OS can reuse those pages in the page tables of multiple processes.

These two processes use **different** pages for heap, stack, and binary...

.. code-block:: python

    ┌────────────────┐         ┌────────────────┐
    │ 9--  →  59-- w │ stack   │ 9--  →  48-- w │ stack
    │ 8--  →  63-- w │ stack   │ 8--  →       - │
    │ 7--  →       - │         │ 7--  →  51-- w │ heap
    │ 6--  →  61-- w │ heap    │ 6--  →  50-- w │ heap
    │ 5--  →  60-- w │ heap    │ 5--  →  46-- w │ heap
    │ 4--  →       - │         │ 4--  →       - │
    │ 3--  →       - │         │ 3--  →  39-- r │ libjson
    │ 2--  →  39-- r │ libjson │ 2--  →  48-- r │ libX11
    │ 1--  →  36-- r │ libc    │ 1--  →  36-- r │ libc
    │ 0--  →  35-- r │ python  │ 0--  →  47-- r │ firefox
    └────────────────┘         └────────────────┘

But they safely share a **single** copy of *libc* and *libjson*

.. code-block:: python

    ┌────────────────┐         ┌────────────────┐
    │                │         │                │
    │                │         │                │
    │                │         │                │
    │                │         │                │
    │                │         │                │
    │                │         │                │
    │                │         │ 3--  →  39-- r │ libjson
    │ 2--  →  39-- r │ libjson │                │
    │ 1--  →  36-- r │ libc    │ 1--  →  36-- r │ libc
    │                │         │                │
    └────────────────┘         └────────────────┘

When we share pages we don't need to use up RAM storing things twice.

Physical RAM page **36** can be reused for every process needing **libc**

Similarly for page **39** and **libjson**, because if you can't write to it
then it looks like it's your own personal copy.


Memory consumed doesn't add up
------------------------------

(06:32 / 36:22 into the video)

This means that the total memory consumed by process A and process B is
**rarely** the sum

(A's memory use) + (B's memory use)

Q: So how can you tell how much memory an additional worker thread or process
will consume on one of your servers?

A: Stop looking at the **quantity** -- “How much RAM does my Python program use?”

Start looking at the **delta**

Δ memory

Ask, “How much memory does each **additional** process / worker / thread cost
(and when will that run me out of RAM)?”

How should you measure that?

*By actual load and resource tests*

Because as we will see, memory usage is *complicated*.


Problem: Python code
====================

(07:37 / 36:22 into the video)

We hit a problem with this idea of sharing binary code.

Q: Where in virtual memory does Python code live?

Python starts running with most RAM pages **shareable**.

.. code-block:: python

    ┌────────────────┐
    │ 9--  →  59-- w │ stack
    │ 8--  →       - │
    │ 7--  →       - │
    │ 6--  →       - │
    │ 5--  →       - │
    │ 4--  →       - │
    │ 3--  →       - │
    │ 2--  →       - │
    │ 1--  →  36-- r │ libc
    │ 0--  →  35-- r │ python
    └────────────────┘

But Python runs :func:`read()` on :file:`foo.py` creating a writable,
**unshareable** page on the heap

.. code-block:: python

    ┌────────────────┐
    │ 9--  →  59-- w │ stack
    │ 8--  →       - │
    │ 7--  →       - │
    │ 6--  →       - │
    │ 5--  →  60-- w │ foo.py   ←
    │ 4--  →       - │
    │ 3--  →       - │
    │ 2--  →       - │
    │ 1--  →  36-- r │ libc
    │ 0--  →  35-- r │ python
    └────────────────┘

Then Python compiles :file:`foo.py` to a code object on *another* writable,
**unshareable** page

.. code-block:: python

    ┌────────────────┐
    │ 9--  →  59-- w │ stack
    │ 8--  →       - │
    │ 7--  →       - │
    │ 6--  →  61-- w │ codeobj  ←
    │ 5--  →  60-- w │ foo.py
    │ 4--  →       - │
    │ 3--  →       - │
    │ 2--  →       - │
    │ 1--  →  36-- r │ libc
    │ 0--  →  35-- r │ python
    └────────────────┘

Code object is another unshareable page.

Only then does :file:`foo.py` start up and create legitimately unique program
data

.. code-block:: python

    ┌────────────────┐
    │ 9--  →  59-- w │ stack
    │ 8--  →       - │
    │ 7--  →  63-- w │ data     ←
    │ 6--  →  61-- w │ codeobj
    │ 5--  →  60-- w │ foo.py
    │ 4--  →       - │
    │ 3--  →       - │
    │ 2--  →       - │
    │ 1--  →  36-- r │ libc
    │ 0--  →  35-- r │ python
    └────────────────┘

The heap winds up as a mix of actual unique data, together with shared code

.. code-block:: python

    ┌────────────────┐
    │ 9--  →  59-- w │ stack
    │ 8--  →       - │
    │ 7--  →  63-- w │ data     ←
    │ 6--  →  61-- w │ codeobj
    │ 5--  →  60-- w │ foo.py
    │ 4--  →       - │
    │ 3--  →       - │
    │ 2--  →       - │
    │ 1--  →  36-- r │ libc
    │ 0--  →  35-- r │ python
    └────────────────┘

Now it *just so happens* that every Python X.Y process loading :file:`foo.py`
will create the *exact same code object*...

--*but the OS does not know about this* because each Python process builds its
code objects separately

To the OS pages that get written in the middle of a process running look like
**heap** and **don't get shared**.

.. code-block:: python

    ┌────────────────┐         ┌────────────────┐
    │ 9--  →  59-- w │    ≠    │ 9--  →  48-- w │ stack
    │ 8--  →       - │         │ 8--  →       - │
    │ 7--  →  63-- w │    ≠    │ 7--  →  71-- w │ data
    │ 6--  →  61-- w │    ≠    │ 6--  →  69-- w │ codeobj
    │ 5--  →  60-- w │    ≠    │ 5--  →  62-- w │ foo.py
    │ 4--  →       - │         │ 4--  →       - │
    │ 3--  →       - │         │ 3--  →       - │
    │ 2--  →       - │         │ 2--  →       - │
    │ 1--  →  36-- r │         │ 1--  →  36-- r │ libc
    │ 0--  →  35-- r │         │ 0--  →  35-- r │ python
    └────────────────┘         └────────────────┘


Q: Why not share code objects? A: reference counts
--------------------------------------------------

(09:12 / 36:22 into the video)

Standard CPython does not support read-only code objects because it constantly
fiddles around with their **reference counts**.

What does this code do?

.. code-block:: python

    f(5)

1. Looks up :func:`f`
2. Gets back a code object
3. Increment its reference count
4. Calls it with ``(5,)``
5. Decrement its reference count

**Why increment the count?**

To prevent other threads from de-allocating and overwriting :func:`f()` while
this thread is busy running it

(10:13 / 36:22 into the video)

.. note::

    Lesson: The OS offers cool optimization when processes need to *share
    code*, but *reference-counted* code objects cannot take advantage of this.

.. seealso::

    http://docs.python.org/extending/extending.html#reference-counts


Dynamic Linking
===============

(10:22 / 36:22 into the video)

How does the Python interpreter find the routines it needs in :file:`libc.so`?

.. code-block:: python

    ┌────────────────┐
    │                │
    │                │
    │                │
    │                │
    │                │
    │                │
    │                │
    │                │
    │ 1--  →  36-- r │ libc.so
    │ 0--  →  35-- r │ python
    └────────────────┘

.. seealso::

    http://en.wikipedia.org/wiki/Dynamic_linking
        Wikipedia article on dynamic linking

    http://en.wikipedia.org/wiki/Shared_libraries#Shared_libraries
        Wikipedia article on shared libraries


The old days: static linking
----------------------------

.. code-block:: bash

     “compile”  “link”

    cmdn.c → cmdn.o ↘
    opts.c → opts.o → prog
    slen.c → slen.o ↗

Every :file:`.o` file has a table of names that it defines and names that it
needs someone else to define

.. code-block:: bash

    $ nm p_move.o
             U _nc_panelhook
             U is_linetouched
    00000000 T move_panel
             U mvwin
             U wtouchln

Linking returns an error if a name needed cannot be found

.. code-block:: bash

    $ ld -o prog foo.o bar.o baz.o

    foo.o: In function `main':
    foo.c:(.text+0x7): undefined reference to `haute'
    collect2: ld returned 1 exit status


:program:`ar` archives
----------------------

(12:32 / 36:22 into the video)

The :program:`ar` tool bundles together related ``.o`` files into ``.a``
archives.

.. code-block:: bash

    $ ar t /usr/lib/libpanel.a
    panel.o
    p_above.o
    p_below.o
    p_bottom.o
    p_delete.o
    p_hide.o
    p_hidden.o
    p_move.o
    p_new.o
    p_replace.o
    p_show.o
    p_top.o
    p_update.o
    p_user.o
    p_win.o

``.a`` files are static libraries.

.. seealso::

    http://en.wikipedia.org/wiki/Static_library
        Wikipedia article on static libraries


Shared libraries
----------------

**Problem:**

(13:43 / 36:22 into the video)

If everyone lets the linker copy :file:`printf.o` into their program,
then memory has to hold many separate copies of each popular library

**Solution:**

The invention of ``.so`` “shared object” files that can be added to the page
table of every program that needs them

.. code-block:: python

    ┌────────────────┐
    │       .        │
    │       .        │
    │       .        │
    │ 1--  →  36-- r │ libc.so
    │ 0--  →  35-- r │ python
    └────────────────┘

The last-minute linking that takes place at runtime to connect python to
:file:`libc.so` is called **Dynamic Linking**

You can see the libraries that a program like python needs with :program:`ldd`:

.. code-block:: bash

    $ ldd /usr/bin/python2.7
          linux-gate.so.1
          libpthread.so.0
          libdl.so.2
          libutil.so.1
          libssl.so.1.0.0
          libcrypto.so.1.0.0
          libz.so.1
          libm.so.6
          libc.so.6
          /lib/ld-linux.so.2

.. note::

    (from Marc) On Mac OS X, use ``otool -L`` to see what libraries a program
    needs:

    .. code-block:: bash

        $ otool -L /usr/bin/python2.6
        /usr/bin/python2.6:
        	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 125.2.1)

    .. seealso::

        `otool <http://developer.apple.com/library/mac/#documentation/Darwin/Reference/ManPages/man1/otool.1.html>`_
            Apple's documentation for the :program:`otool` program.

You can see the specific names it needs with :program:`nm` (“names”)

.. code-block:: bash

    $ nm -D /usr/bin/python2.7
            ...
            U unlink
            U unsetenv
            U utime
            U utimes
            U wait
            U wait3
            U wait4
            U waitpid
            U wcscoll
            ...

.. note::

    (from Marc) ``-D`` (alias: ``--dynamic``) is an option of :program:`nm`
    from `GNU binutils <http://www.gnu.org/software/binutils/>`_. The
    :program:`nm` in Mac OS X
    does not have it.

Not only is it usual for the Python *binary* to be dynamically linked but
*Python extension modules* sometimes link against shared libraries too

The :mod:`lxml.etree` Python module is famous for this:

.. code-block:: bash

    $ ldd /usr/lib/pyshared/python2.7/lxml/etree.so
           linux-gate.so.1
           libxslt.so.1
           libexslt.so.0
           libxml2.so.2
           libpthread.so.0
           libc.so.6
           libm.so.6
           libgcrypt.so.11
           libdl.so.2
           libz.so.1
           /lib/ld-linux.so.2
           libgpg-error.so.0

.. seealso::

    http://en.wikipedia.org/wiki/Shared_libraries#Shared_libraries
        Wikipedia article on shared libraries

    http://en.wikipedia.org/wiki/Dynamic_linking
        Wikipedia article on dynamic linking


Shared library problems
-----------------------

Shared library: :file:`libtiny.so`

.. code-block:: c

    /* libtiny.c */
    helper() { return 42; }

.. code-block:: bash

    gcc libtiny.c -shared -o libtiny.so

Python module: :file:`tinymodule.so`

.. code-block:: c

    /* tinymodule.c */

    #include <python2.7/Python.h>

    static PyMethodDef TinyMethods[] = {
        {NULL, NULL, 0, NULL}
    };

    PyMODINIT_FUNC inittiny(void)
    {
        helper();  /* the call into libtiny.so */
        (void) Py_InitModule("tiny", TinyMethods);
    }

.. code-block:: bash

    gcc tinymodule.c -L. -ltiny -shared -o tinymodule.so

Having created these two :file:`.so` files we need to add the current directory
to the OS shared library search path

.. code-block:: bash

    $ export LD_LIBRARY_PATH=.

.. note:: The above is for Linux. On Mac OS X, use ``DYLD_LIBRARY_PATH``

So the module :file:`tinymodule.so` needs the library :file:`libtiny.so`

.. code-block:: bash

    $ ldd tinymodule.so
           linux-gate.so.1
           libtiny.so
           libc.so.6
           /lib/ld-linux.so.2

.. note:: The above is for Linux. On Mac OS X, instead of :program:`ldd`, use :program:`otool -L`.

**Shared library present, compatible**

.. code-block:: bash

    $ ls
    libtiny.c   tinymodule.c
    libtiny.so  tinymodule.so

    $ python -c 'import tiny'

    $ echo $?
    0

**Shared library missing**

.. code-block:: bash

    $ rm libtiny.so
    $ python -c 'import tiny'
    ImportError: libtiny.so:
        cannot open shared object file:
        No such file or directory

**Incompatible shared library**

.. code-block:: c

    /* libtiny.c */
    different_helper() { return 42; }
    gcc libtiny.c -shared -o libtiny.so

.. code-block:: bash

    $ python -c 'import tiny'
    ImportError: ./tinymodule.so:
        undefined symbol: helper

*“Cannot open shared object”* and
*“undefined symbol”* are possible because OS tries to link a binary
(:file:`tinymodule.so`) to its dependencies (:file:`libtiny.so`) *at runtime*


Demand Paging
=============

(18:00 / 36:22 into the video)

The OS does not load pages from disk until your program reads or writes from
them

Imagine a program that has just started running

Files:

* python — 3 pages
* libc.so — 2 pages

At first, only one page of the python binary is loaded

.. code-block:: bash

    ┌────────────────┐
    │ 9--  →  59-- w │  stack
    │ 8--  →       - │
    │ 7--  →       - │
    │ 6--  →  61-- w │  heap
    │ 5--  →       - │
    │ 4--  →       - │ (libc.so)
    │ 3--  →       - │ (libc.so)
    │ 2--  →       - │ (python)
    │ 1--  →       - │ (python)
    │ 0--  →  35-- r │  python ← LOAD FROM DISK
    └────────────────┘

Then Python tries to run the instruction at **212** so that page gets loaded
too

.. code-block:: bash

    ┌────────────────┐
    │ 9--  →  59-- w │  stack
    │ 8--  →       - │
    │ 7--  →       - │
    │ 6--  →  61-- w │  heap
    │ 5--  →       - │
    │ 4--  →       - │ (libc.so)
    │ 3--  →       - │ (libc.so)
    │ 2--  →  37-- r │  python ← LOAD FROM DISK
    │ 1--  →       - │ (python)
    │ 0--  →  35-- r │  python
    └────────────────┘

Then Python calls a function at libc offset **178** thus **300 + 178 = 478**

.. code-block:: bash

    ┌────────────────┐
    │ 9--  →  59-- w │  stack
    │ 8--  →       - │
    │ 7--  →       - │
    │ 6--  →  61-- w │  heap
    │ 5--  →       - │
    │ 4--  →  41-- r │  libc.so ← LOAD FROM DISK
    │ 3--  →       - │ (libc.so)
    │ 2--  →  37-- r │  python
    │ 1--  →       - │ (python)
    │ 0--  →  35-- r │  python
    └────────────────┘

If Python keeps calling the same routines, then only those 3 pages ever get
loaded

So pages are loaded from disk **lazily**

Heap allocation works the **same way**

(18:50 / 36:22 into the video)

**Q: How many RAM pages are allocated if you ask the OS for three new memory pages?**

**A: None!**

The OS allocates *no pages* but simply makes an adjustment to its
record-keeping

.. code-block:: bash

    ┌────────────────┐
    │ 9--  →  59-- w │  stack
    │ 8--  →       - │
    │ 7--  →       - │ (heap)  ←
    │ 6--  →       - │ (heap)  ←
    │ 5--  →       - │ (heap)  ←
    │ 4--  →  61-- w │  heap
    │ 3--  →       - │
    │ 2--  →  41-- r │  libc.so
    │ 1--  →       - │ (python)
    │ 0--  →  35-- r │  python
    └────────────────┘

The OS will now respond to page faults at **500–799** by allocating a new page,
not raising a Segmentation Fault

But no pages are actually allocated until a **read** or **write** shows they
are *needed*

(19:19 / 36:22 into the video)

**That is the difference**

between **VIRT** and **RES** when you run :program:`top`

**top**

.. code-block:: bash

                   ↓     ↓
    
      PID USER    VIRT  RES  SHR S %CPU %MEM COMMAND
     1217 root    195m  94m  53m S    2  2.6 Xorg
     7304 brandon 183m  45m  19m S    2  1.3 chromium
     2027 brandon 531m 162m  37m S    1  4.5 chromium
     2319 brandon 179m  58m  23m S    1  1.6 chromium
    18841 brandon 2820 1188  864 R    1  0.0 top
     9776 brandon 175m  49m  12m S    0  1.4 chromium
    
                   ↑     ↑

.. seealso::

    `Demand paging <http://en.wikipedia.org/wiki/Demand_paging>`_
        Wikipedia article on Demand Paging

**VIRT**

* Virtual Image
* All memory pages that promise not to return a Segmentation Fault

**RES**

* Resident Size
* Only memory pages that have actually been allocated

**Diagram to illustrate the difference**

* VIRT — 8 pages
* RES — 4 pages

.. code-block:: bash

    ┌────────────────┐         VIRT  RES
    │ 9--  →  59-- w │  stack    ✓    ✓
    │ 8--  →       - │
    │ 7--  →       - │ (heap)    ✓
    │ 6--  →       - │ (heap)    ✓
    │ 5--  →       - │ (heap)    ✓
    │ 4--  →  61-- w │  heap     ✓    ✓
    │ 3--  →       - │
    │ 2--  →  41-- r │  libc.so  ✓    ✓
    │ 1--  →       - │ (python)  ✓
    │ 0--  →  35-- r │  python   ✓    ✓
    └────────────────┘

Only RES can actually run you out of memory

Look at RES when you wonder where all of your RAM is going

.. code-block:: bash

                         ↓
    
      PID USER    VIRT  RES  SHR S %CPU %MEM COMMAND
     1217 root    195m  94m  53m S    2  2.6 Xorg
     7304 brandon 183m  45m  19m S    2  1.3 chromium
     2027 brandon 531m 162m  37m S    1  4.5 chromium
     2319 brandon 179m  58m  23m S    1  1.6 chromium
    18841 brandon 2820 1188  864 R    1  0.0 top
     9776 brandon 175m  49m  12m S    0  1.4 chromium
    
                         ↑


:file:`/proc/<pid>/smaps`
-------------------------

(20:19 / 36:22 into the video)

Linux file; shows whether physical RAM pages have been allocated to a segment

.. note::

    (from Marc) This is on Linux; OS X does not have the :file:`/proc`
    filesystem. The OS X equivalent of this is the :program:`vmmap` command.

    .. seealso::

        `vmmap(1) <https://developer.apple.com/library/mac/#documentation/Darwin/Reference/ManPages/man1/vmmap.1.html>`_
            Man page for the :program:`vmmap` program.

        `"Viewing Virtual Memory Usage" <http://developer.apple.com/library/mac/#documentation/Performance/Conceptual/ManagingMemory/Articles/VMPages.html>`_
            A section from the `Memory Usage Performance Guidelines guide
            <http://developer.apple.com/library/mac/#documentation/Performance/Conceptual/ManagingMemory/ManagingMemory.html>`_
            in the `Mac OS X Developer Library <http://developer.apple.com/library/mac/navigation/>`_

Here are some segments of a new python process sitting quietly at its prompt

.. code-block:: bash

    08048000-08238000 r-xp .../bin/python
    Size:               1984 kB
    Rss:                 896 kB
    
    b73b9000-b752f000 r-xp .../libc-2.13.so
    Size:               1496 kB
    Rss:                 528 kB
    
    bfc48000-bfc69000 rw-p [stack]
    Size:                136 kB
    Rss:                 104 kB

**Demand Paging: Example**

Calling a Python function that was not called as Python started up

.. code-block:: bash

    08048000-08238000 r-xp .../bin/python
    Size:               1984 kB
    Rss:                 896 kBA
    
.. code-block:: python

    >>> frozenset()
    
.. code-block:: bash

    08048000-08238000 r-xp .../bin/python
    Size:               1984 kB
    Rss:                 916 kB

**916 - 896 = 20** new kilobytes

So invoking new sections of a binary or library pulls more pages into RAM

*But* since binary and library code can always be reloaded from disk, the OS
can also *discard* those pages

.. code-block:: bash

    $ python -c 'range(500000000)'

This allocates lots of memory As it runs, your OS will dump information
overboard to reclaim every RAM page it can

What did this memory hog do to the other Python process that was sitting
quietly at its prompt?

.. code-block:: bash

    08048000-08238000 r-xp .../bin/python
    Size:               1984 kB
    Rss:                 916 kB
                         ↑
                         before the hog ran

                         after
                         ↓
    08048000-08238000 r-xp .../bin/python
    Size:               1984 kB
    Rss:                 464 kB

**916 - 464 = 452k** were thrown overboard!

Those 452k pages will be re-loaded, on-demand, from disk if this Python process
tries to access them again

The process will slow down as de-allocated pages are re-loaded

* L1 cache — 3s grabbing a piece of paper
* L2 cache — 14s picking a book from a shelf
* System RAM — 4-minute walk down the hall
* Hard drive seek —

  “like leaving the building to roam the earth for one year
  and three months.”

    — Gustavo Duarte


Forking
=======

(22:35 / 36:22 into the video)

On primitive operating systems the only way to create a new process is to start
a whole new program

Linux and OS X support :func:`fork()`!

:func:`fork()` creates a child process that continues on from the parent's
current state

.. code-block:: bash

    Process 1  A
               ↓
               B
               ↓
               C
               ↓
               D — os.fork() ↴  Process 2
               ↓             ↓
               E             E
               ↓             ↓
               F             F
               ↓             ↓

The OS *could* implement :func:`fork()` naively by copying every writeable page
in the parent so that the child had its own copy

.. code-block:: bash

      fork() parent              fork() child
    ┌────────────────┐         ┌─────────────────┐
    │ 9--  →  59-- w │ stack   │ 9--  → new copy │
    │ 8--  →  63-- w │ stack   │ 8--  → new copy │
    │ 7--  →       - │         │ 7--  →        - │
    │ 6--  →  61-- w │ heap    │ 6--  → new copy │
    │ 5--  →  60-- w │ heap    │ 5--  → new copy │
    │ 4--  →       - │         │ 4--  →        - │
    │ 3--  →       - │         │ 3--  →        - │
    │ 2--  →  39-- r │ libjson │ 2--  →  (same)  │
    │ 1--  →  36-- r │ libc    │ 1--  →  (same)  │
    │ 0--  →  35-- r │ python  │ 0--  →  (same)  │
    └────────────────┘         └─────────────────┘

But immediately copying every page could take a long time for a large process,
and during the copy both parent and child would be hung waiting!

Instead, as you might guess, the OS implements :func:`fork()`

**lazily,**

copying pages on-demand when the parent or child performs a write

So the OS starts :func:`fork()` by copying the page table, marking all pages
read-only in both processes, then letting them keep running

.. code-block:: bash

      fork() parent              fork() child
    ┌────────────────┐         ┌────────────────┐
    │ 9--  →  59-- r │ stack   │ 9--  →  59-- r │
    │ 8--  →  63-- r │ stack   │ 8--  →  63-- r │
    │ 7--  →       - │         │ 7--  →       - │
    │ 6--  →  61-- r │ heap    │ 6--  →  61-- r │
    │ 5--  →  60-- r │ heap    │ 5--  →  60-- r │
    │ 4--  →       - │         │ 4--  →       - │
    │ 3--  →       - │         │ 3--  →       - │
    │ 2--  →  39-- r │ libjson │ 2--  →  (same) │
    │ 1--  →  36-- r │ libc    │ 1--  →  (same) │
    │ 0--  →  35-- r │ python  │ 0--  →  (same) │
    └────────────────┘         └────────────────┘

As writes cause page faults the OS makes copies.  Here page **63** is copied to
**77** and marked **w**.

.. code-block:: bash

        fork() parent                  fork() child
      ┌────────────────┐            ┌────────────────┐
      │ 9--  →  59-- r │   stack    │ 9--  →  59-- r │
    → │ 8--  →  63-- w │   stack    │ 8--  →  77-- w │ ←
      │ 7--  →       - │            │ 7--  →       - │
      │ 6--  →  61-- r │   heap     │ 6--  →  61-- r │
      │ 5--  →  60-- r │   heap     │ 5--  →  60-- r │
      │ 4--  →       - │            │ 4--  →       - │
      │ 3--  →       - │            │ 3--  →       - │
      │ 2--  →  39-- r │   libjson  │ 2--  →  (same) │
      │ 1--  →  36-- r │   libc     │ 1--  →  (same) │
      │ 0--  →  35-- r │   python   │ 0--  →  (same) │
      └────────────────┘            └────────────────┘

(24:17 / 36:22 into the video)

Q: How well does this *usually* work?

A: It works *great*!

Only the *differences* that develop between the parent and child memory images
incur additional storage

Q: But how does it work for *Python*?

A: Terribly!

Thanks to reference counts, merely *glancing* at data with CPython forces the
OS to create a separate copy


CPython vs. PyPy
================

.. code-block:: python

    # Create a list

    biglist = ['foo']
    for n in range(1, 8460000):
        biglist.append(n)   # ~100 MB

    # put fork() here

    # Iterate across the list

    t0 = time.time()
    all(n for n in biglist)
    t1 = time.time()


CPython
-------

**Before C Python iterates**

.. code-block:: bash

    $ cat /proc/22364/smaps | awk '/heap/,/Private_D/'
    08d60000-0ef90000 rw-p 00000000 00:00 0     [heap]
    Size:             100544 kB
    Rss:              100544 kB
    Pss:               50294 kB
    Shared_Clean:          0 kB
    Shared_Dirty:     100500 kB
    Private_Clean:         0 kB
    Private_Dirty:        44 kB

**After C Python iterates**

.. code-block:: bash

    $ cat /proc/22364/smaps | awk '/heap/,/Private_D/'
    08d60000-0ef90000 rw-p 00000000 00:00 0     [heap]
    Size:             100544 kB
    Rss:              100544 kB
    Pss:              100274 kB
    Shared_Clean:          0 kB
    Shared_Dirty:        540 kB
    Private_Clean:         0 kB
    Private_Dirty:    100004 kB

At finish: **0.5%** of heap shared


PyPy 2.8
--------

**Before PyPy iterates**

.. code-block:: bash

    $ cat /proc/22385/smaps | awk '/heap/,/Private_D/'
    0a5c9000-11fba000 rw-p 00000000 00:00 0     [heap]
    Size:             124868 kB
    Rss:              124540 kB
    Pss:               62274 kB
    Shared_Clean:          0 kB
    Shared_Dirty:     124532 kB
    Private_Clean:         0 kB
    Private_Dirty:         8 kB

**After PyPy iterates**

.. code-block:: bash

    $ cat /proc/22385/smaps | awk '/heap/,/Private_D/'
    0a5c9000-11fba000 rw-p 00000000 00:00 0     [heap]
    Size:             124868 kB
    Rss:              124868 kB
    Pss:               63160 kB
    Shared_Clean:          0 kB
    Shared_Dirty:     123416 kB
    Private_Clean:         0 kB
    Private_Dirty:      1452 kB

At finish: **96.1%** of heap shared

(26:01 / 36:22 into the video)

**Lesson:**

Forked worker processes in Unix *share* memory when the parent process builds
*read-only* data structures

—**but** not in CPython

.. seealso::

    http://en.wikipedia.org/wiki/Fork_(operating_system)
        Wikipedia article on process forking


Explicitly Sharing Memory
=========================

(26:14 / 36:22 into the video)

How can two Python procedures share *writeable* memory to collaborate?

1. Threads
2. Memory maps


Threads
-------

(26:37 / 36:22 into the video)

Creating a thread is like :func:`fork()` except that the heap *remains shared*
between the two threads of control

Python supports threads on both Unix and Windows

.. code-block:: python

    import threading
    t = threading.Thread(target=myfunc)
    t.start()

Each thread gets its *own stack* so the two threads can call different
functions, but *all other* data structures are *shared*

.. code-block:: bash

        main thread               child thread
    ┌────────────────┐         ┌────────────────┐
    │ 9--  →  59-- r │  stack  │ 9--  →  59-- r │
    │ 8--  →  63-- w │  stack  │ 8--  →  77-- w │
    └────────────┬───┴─────────┴──┬─────────────┘
                 │ 7--  →       - │
                 │ 6--  →  61-- w │ heap
                 │ 5--  →  60-- w │ heap
                 │ 4--  →       - │
                 │ 3--  →       - │
                 │ 2--  →  39-- r │ libjson
                 │ 1--  →  36-- r │ libc
                 │ 0--  →  35-- r │ python
                 └────────────────┘

Since threads share *every* data structure they have to be *very* careful

.. seealso::

    http://en.wikipedia.org/wiki/Thread_(computing)
        Wikipedia article on threads


Memory maps
-----------

(27:19 / 36:22 into the video)

Using :func:`mmap()` a parent process can create shared memory that will be
inherited by all the child workers it forks

.. code-block:: python

    import mmap, os

    map = mmap.mmap(-1, 100)
    os.fork()
    ...

.. code-block:: bash

     fork() parent                 fork() child
    ┌────────────────┐           ┌────────────────┐
    │ 9--  →  59-- r │   stack   │ 9--  →  59-- r │
    │ 8--  →  63-- w │   stack   │ 8--  →  77-- w │
    └─────────────┬──┴───────────┴─┬──────────────┘
                  │ 7--  →  88-- - │ mmap segment
    ┌─────────────┴──┬───────────┬─┴──────────────┐
    │ 6--  →  61-- r │   heap    │ 6--  →  61-- r │
    │ 5--  →  60-- r │   heap    │ 5--  →  60-- r │
    │ 4--  →       - │           │ 4--  →       - │
    │ 3--  →       - │           │ 3--  →       - │
    │ 2--  →  39-- r │   libjson │ 2--  →  (same) │
    │ 1--  →  36-- r │   libc    │ 1--  →  (same) │
    │ 0--  →  35-- r │   python  │ 0--  →  (same) │
    └────────────────┘           └────────────────┘

This supports very fast RAM-based communication between processes

*without* requiring every data structure on the heap to carry *locks* or other
protection

**Another mmap() capability**

Remember how the OS loads pages from binary programs like :file:`python` and
shared libraries like :file:`libc.so` on-demand, not all at once?

Well, with :func:`mmap()` you can do that yourself, with normal files!

.. code-block:: python

    mmap(myfile.fileno(), 0)

lets you replace :file:`seek()`, :file:`read()`, and :file:`write()` calls and
simply access it like an array!

.. code-block:: bash

    ┌────────────────┐
    │ 9--  →  59-- w │  stack
    │ 8--  →       - │
    │ 7--  →  91-- - │  mmap[1] ↔ myfile
    │ 6--  →  90-- - │  mmap[0] ↔ myfile
    │ 5--  →       - │
    │ 4--  →  61-- w │  heap
    │ 3--  →       - │
    │ 2--  →  41-- r │  libc.so
    │ 1--  →       - │ (python)
    │ 0--  →  35-- r │  python
    └────────────────┘

.. seealso::

    http://en.wikipedia.org/wiki/Mmap
        Wikipedia article on :func:`mmap`


Summary
=======

(28:40 / 36:22 into the video)

Your processes all access memory through the mediation of a *page table*

*Page tables* power all kinds of fun RAM optimizations:

1. Demand loading
2. Shared libraries
3. Memory maps
4. :func:`fork()`
5. Threads

And

*PyPy* lacks reference counts which lets the OS *do its magic* and *conserve
resources*

(29:16 / 36:22 into the video)


Questions
=========

(29:35 / 36:22 into the video)

Just one question

* Erik Rose said he recently started using :func:`mmap()` in a non-Python
  context.  Free RAM meters (e.g.: :program:`free`, :program:`top`) behaving
  strangely. Do you know if mmapped memory that is actually resident affects
  free counts or does it somehow go behind its back?

  - A: Very dependent on the version of the operating system.
       Reporting tools can have odd effects.
       What I would expect is for it only show pages disappearing when they're allocated.
       When you mmap, you can ask for it to eagerly grab the memory.


Marc's Prologue
===============

Resources
---------

* A great free resource is `Ulrich Drepper's DSO How To (a.k.a.:
  "How To Write Shared Libraries")
  <http://www.akkadia.org/drepper/dsohowto.pdf>`_.

* `An awesomely detailed blog post on Position-Independent Code (PIC) by Eli
  Bendersky
  <http://eli.thegreenplace.net/2011/11/03/position-independent-code-pic-in-shared-libraries/>`_

* For more great info on this kind of stuff, check out `"Linkers & Loaders" by
  John R. Levine
  <http://www.amazon.com/gp/product/1558604960/ref=as_li_qf_sp_asin_il?ie=UTF8&tag=marcsepinion-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=1558604960>`_.

    .. raw:: html

        <a href="http://www.amazon.com/gp/product/1558604960/ref=as_li_qf_sp_asin_il?ie=UTF8&tag=marcsepinion-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=1558604960"><img border="0" src="http://ws.assoc-amazon.com/widgets/q?_encoding=UTF8&Format=_SL160_&ASIN=1558604960&MarketPlace=US&ID=AsinImage&WS=1&tag=marcsepinion-20&ServiceVersion=20070822" ></a><img src="http://www.assoc-amazon.com/e/ir?t=marcsepinion-20&l=as2&o=1&a=1558604960" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

    The book also has a `web site <http://linker.iecc.com/>`_.

* To delve deep into how the Linux kernel implements virtual memory (and other
  stuff), you might check out `"Linux Kernel Development" by Robert Love
  <http://www.amazon.com/gp/product/0672329468/ref=as_li_qf_sp_asin_il?ie=UTF8&tag=marcsepinion-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0672329468>`_

    .. raw:: html

        <a href="http://www.amazon.com/gp/product/0672329468/ref=as_li_qf_sp_asin_il?ie=UTF8&tag=marcsepinion-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0672329468"><img border="0" src="http://ws.assoc-amazon.com/widgets/q?_encoding=UTF8&Format=_SL160_&ASIN=0672329468&MarketPlace=US&ID=AsinImage&WS=1&tag=marcsepinion-20&ServiceVersion=20070822" ></a><img src="http://www.assoc-amazon.com/e/ir?t=marcsepinion-20&l=as2&o=1&a=0672329468" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

