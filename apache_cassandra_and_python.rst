********************************************************************************************************
Apache Cassandra and Python
********************************************************************************************************

**Presenters**: `Jeremiah Jordan <https://us.pycon.org/2012/speaker/profile/150/>`_ from Morningstar, Inc.

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/122/

**Slides**: http://goo.gl/8Byd8

**Video**: https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1

**Video running time**: 31:00


Why are you here?
=================

`(00:24) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1>`_

* You were too lazy to get out of your seat.
* Someone said "NoSQL".
* You want to learn about using Cassandra from Python.


What am I going to talk about?
==============================

`(00:34) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=00m34s>`_

* What is Cassandra
* Starting up a local dev/unit test instance
* Using Cassandra from Python
* Indexing/Schema design


What am I not going to talk about?
----------------------------------

`(00:55) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=00m34s>`_

* Setting up and maintaining a production cluster



Where can I get the slides?
---------------------------

`(01:07) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=01m07s>`_

http://goo.gl/8Byd8

points to

http://www.slideshare.net/jeremiahdjordan/pycon-2012-apache-cassandra


What is Apache Cassandra (Buzz Word description)?
-------------------------------------------------

`(01:23) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=01m23s>`_

"Cassandra is a highly scalable, eventually consistent, distributed, structured
key-value store. Cassandra brings together the distributed systems technologies
from `Dynamo
<http://s3.amazonaws.com/AllThingsDistributed/sosp/amazon-dynamo-sosp2007.pdf>`_
and the data model from `Google's BigTable
<http://research.google.com/archive/bigtable-osdi06.pdf>`_. Like Dynamo,
Cassandra is `eventually consistent
<http://www.allthingsdistributed.com/2008/12/eventually_consistent.html>`_.
Like BigTable, Cassandra provides a ColumnFamily-based data model richer than
typical key/value systems."

From the Cassandra Wiki: http://wiki.apache.org/cassandra/


What is Apache Cassandra?
-------------------------

`(01:58) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=01m58s>`_

* Column based key value store (multi level dictionary)
* Combination of Dynamo (Amazon) and BigTable (Google)
* Schema-optional


Basic structure of data in Cassandra
------------------------------------

`(02:26) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=02m26s>`_

.. image:: http://cl.ly/3o1A1j2H3s1P1O1m3w24/Screen%20shot%202012-03-29%20at%2010.30.38%20PM.png

* A *keyspace* is kind of like a schema in a relational database.
* A *column family* is kind of like a table in a relational database.

`(02:46) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=02m46s>`_

.. image:: http://cl.ly/461h461q0u1R2Y2E392r/Screen%20shot%202012-04-01%20at%209.00.12%20AM.png

* Keys are in a random order; the recommended way of using Cassandra; allows distributing things evenly over your cluster
* With ordered keys, you have to be careful not to create hotspots in your cluster
* Column names are sorted in alphabetic order
* You can optionally type values
* Every value has a timestamp associated with it; used for conflict resolution -- make sure that clocks are in sync.


Multi-level Dictionary
----------------------

`(05:08) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=05m08s>`_

.. code-block:: json

   Column Family
        |       Columns
        ↓           |
    {"UserInfo":    ↓
        {"John": {"age": 32,
           ↑      "email": "john@gmail.com",
           |      "gender": "M",
           |      "state": "IL"}}}
           |                ↑
          Key               |
                          Values


Well, really an *ordered* dictionary
------------------------------------

.. code-block:: python

    {"UserInfo":
        {"John":
            OrderedDict(
                [("age", 32),
                 ("email", "john@gmail.com"),
                 ("gender", "M"),
                 ("state", "IL")])}}


Where do I get it?
------------------

`(05:30) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=05m30s>`_

From the Apache Cassandra project:

http://cassandra.apache.org/

or DataStax hosts some Debian and RedHat packages:

http://www.datastax.com/docs/1.0/install


How do I run it?
----------------

`(05:47) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=05m47s>`_

Edit :file:`conf/cassandra.yaml`:

    * Change data/commit log locations
    * defaults: :file:`/var/cassandra/data` and :file:`/var/cassandra/commitlog`

Edit :file:`conf/log4j-server.properties`:

    * Change the log location/levels
    * default: :file:`/var/log/cassandra/system.log`

`(06:10) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=06m10s>`_

Edit :file:`conf/cassandra-env.sh` (:file:`bin/cassandra.bat` on Windows)

    * Update JVM memory usage
    * default: 1/2 your RAM

.. code-block:: bash

    $ ./cassandra -f     # -f means launch in foreground


Setup up tips for local instances
---------------------------------

`(06:43) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=06m43s>`_

Make templates out of :file:`cassandra.yaml` and :file:`log4j-server.properties`

Update :file:`cassandra` script to generate the actual files

(run them through :program:`sed` or something).


Server is running, what now?
----------------------------

`(07:10) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=07m10s>`_

.. code-block:: bash

    $ ./cassandra-cli
    connect localhost/9160;

    create keyspace ApplicationData
        with placement_strategy =
            'org.apache.cassandra.locator.SimpleStrategy'
        and strategy_options =
            [{replication_factor:1}];

`(08:20) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=08m20s>`_

.. code-block:: bash

    use ApplicationData;

    create column family UserInfo
        and comparator = 'AsciiType';

    create column family ChangesOverTime
        and comparator = 'TimeUUIDType';


Connect from Python
-------------------

http://wiki.apache.org/cassandra/ClientOptions

Thrift - See the "interface" directory (Do not use!!!)

`Pycassa <http://pypi.python.org/pypi/pycassa/>`_ - ``pip install pycassa`` -- the one we'll talk about today

`Telephus (twisted) <https://github.com/driftx/Telephus>`_ - ``pip install telephus``

`DB-API 2.0 <http://code.google.com/a/apache-extras.org/p/cassandra-dbapi2/>`_ (CQL) - ``pip install cassandra-dbapi2``


Thrift (don't use it)
---------------------

`(10:48) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=10m48s>`_

.. image:: http://cl.ly/3i0S2N0p3k0I0S3t2a0d/Screen%20shot%202012-04-01%20at%2010.00.38%20AM.png


Pycassa
-------

`(10:55) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=10m55s>`_

.. code-block:: python

    import pycassa
    from pycassa.pool import ConnectionPool
    from pycassa.columnfamily import ColumnFamily

    pool = ConnectionPool('ApplicationData',
                          ['localhost:9160'])
    col_fam = ColumnFamily(pool, 'UserInfo')
    col_fam.insert('John', {'email': 'john@gmail.com'})


http://pycassa.github.com/pycassa/

http://github.com/twissandra/twissandra/ -- An example application; Twitter clone using Django and Pycassa


Connect
-------

`(11:22) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=11m22s>`_

.. code-block:: python

    """                      Keyspace
                                 |
                                 ↓                 """
    pool = ConnectionPool('ApplicationData',
                          ['localhost:9160'])
    """                          ↑
                                 |
                             Server list           """

Cassandra scales very linearly. Netflix has some nice papers online about it.


Open Column Family
------------------

`(12:38) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=12m38s>`_

.. code-block:: python

    """                   Connection Pool
                             |
                             ↓                 """
    col_fam = ColumnFamily(pool, 'UserInfo')
    """                             ↑
                                    |
                                Column Family  """


Write
-----

`(12:50) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=12m50s>`_

.. code-block:: python

    col_fam.insert('John', {'email': 'john@gmail.com'})


Read
----

`(13:03) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=13m03s>`_

.. code-block:: python

    readData = col_fam.get('John',
                           columns=['email'])


Delete
------

`(13:18) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=13m18s>`_

.. code-block:: python

    col_fam.remove('John',
                   columns=['email'])

Batch
-----

`(13:23) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=13m23s>`_

.. code-block:: python

    col_fam.batch_insert(
        {'John': {'email': 'john@gmail.com',
                  'state': 'IL',
                  'gender': 'M'},
         'Jane': {'email': 'jane@python.org',
                  'state': 'CA'
                  'gender': 'M'}})

Batch (streaming)
-----------------

`(13:44) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=13m44s>`_

.. code-block:: python

    b = col_fam.batch(queue_size=10)

    b.insert('John',
             {'email': 'john@gmail.com',
              'state': 'IL',
              'gender': 'M'})

    b.insert('Jane',
             {'email': 'jane@python.org',
              'state': 'CA'})

    b.remove('John', ['gender'])
    b.remove('Jane')
    b.send()


Batch (Multi-CF)
----------------

`(14:39) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=14m39s>`_

.. code-block:: python

    from pycassa.batch import Mutator
    import uuid

    b = Mutator(pool)

    b.insert(col_fam,
             'John', {'gender': 'M'})

    b.insert(index_fam,
             '2012-03-09',
             {uuid.uuid1().bytes:
                    'John:gender:F:M'})


Batch Read
----------

`(15:28) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=15m28s>`_

.. code-block:: python

    readData = col_fam.multiget(['John', 'Jane', 'Bill'])


Column Slice
------------

`(15:42) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=15m42s>`_

.. code-block:: python

    d = col_fam.get('Jane',
                    column_start='email',
                    column_finish='state')

    d = col_fam.get('Bill',
                    column_reversed = True,
                    column_count=2)

    startTime = pycassa.util.convert_time_to_uuid(time.time() - 600)

    d = index_fam.get('2012-03-31',
                      column_start=startTime,
                      column_count=30)


Types
-----

`(17:00) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=17m00s>`_

.. code-block:: python

    from pycassa.types import *

    col_fam.column_validators['age'] = IntegerType()

    col_fam.column_validators['height'] = FloatType()

    col_fam.insert('John', {'age': 32, 'height': 6.1})


Column Family Map
-----------------

`(17:52) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=17m52s>`_

.. code-block:: python

    from pycassa.types import *

    class User(object):
        key = Utf8Type()
        email = AsciiType()
        age = IntegerType()
        height = FloatType()
        joined = DateType()

    # `(18:48) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=18m48s>`_

    from pycassa.columnfamilymap import ColumnFamilyMap

    cfmap = ColumnFamilyMap(User, pool, 'UserInfo')


Write
-----

`(19:05) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=19m05s>`_

.. code-block:: python

    from datetime import datetime

    user = User()
    user.key = 'John'
    user.email = 'john@gmail.com'
    user.age = 32
    user.height = 6.1
    user.joined = datetime.now()
    cfmap.insert(user)


Read/Delete
-----------

`(19:37) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=19m37s>`_

.. code-block:: python

    user = cfmap.get('John')

    users = cfmap.multiget(['John', 'Jane'])

    cfmap.remove(user)


Timestamps/consistency
----------------------

`(20:09) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=19m37s>`_

.. code-block:: python

    col_fam.read_consistency_level = ConsistencyLevel.QUORUM
    col_fam.write_consistency_level = ConsistencyLevel.ONE

    col_fam.get('John',
                read_consistency_level=ConsistencyLevel.ONE)

    col_fam.get('John',
                include_timestamp=True)

A quorum is n / 2 nodes.


Indexing
--------

`(22:00) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=22m00s>`_

Native secondary indexes

Roll your own with wide rows


Indexing Links
--------------

`(22:09) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=22m09s>`_

Intro to indexing

* http://www.datastax.com/dev/blog/whats-new-cassandra-07-secondary-indexes

Blog post and presentation going through some options

* http://www.anuff.com/2011/02/indexing-in-cassandra.html
* http://www.slideshare.net/edanuff/indexing-in-cassandra

Another blog post describing different patterns for indexing

* http://pkghosh.wordpress.com/2011/03/02/cassandra-secondary-index-patterns


Native Indexes
--------------

`(21:20) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=22m20s>`_

Easy to add, just update the schema

Can use filtering queries

Not recommended for high cardinality values (i.e.: timestamps, birth dates, keywords, etc.)

Makes writes slower to indexed columns


Add index
---------

`(23:29) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=23m29s>`_

.. code-block:: sql

    update column family UserInfo
        with column_metadata = [
            {column_name: state,
             validation_class: UTF8Type,
             index_type: KEYS};


Native indexes
--------------

`(23:52) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=23m52s>`_

.. code-block:: python

    from pycassa.index import *

    state_expr = create_index_expression('state', 'IL')
    age_expr = create_index_expression('age', 20, GT)
    clause = create_index_clause([state_expr, age_expr], count=20)

    for key, userInfo in col_fam.get_indexed_slices(clause):
        # Do stuff


Rolling your own
----------------

`(24:40) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=24m40s>`_

Removing changed values yourself

Know the new value doesn't exist, no read before write

Index can be denormalized query, not just an index

Can use things like composite columns and other tricks


Questions
---------

`(25:13) <https://www.youtube.com/watch?v=188mXjwdkak&feature=autoplay&list=PLBC82890EA0228306&lf=plpp_video&playnext=1#t=25m13s>`_



