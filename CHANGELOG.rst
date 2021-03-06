What’s new in django-cachalot?
==============================

1.2.1
-----

**Mandatory update if you’re using django-cachalot 1.2.0.**

This version reverts the cache keys hashing change from 1.2.0,
as it was leading to a non-shared cache when Python used a random seed
for hashing, which is the case by default on Python 3.3, 3.4, & 3.5,
and also on 2.7 & 3.2 if you set ``PYTHONHASHSEED=random``.

1.2.0
-----

**WARNING: This version is unsafe, it can lead to invalidation errors**

- Adds Django 1.9 support
- Simplifies and speeds up cache keys hashing
- Documents how to use django-cachalot with a replica database
- Adds ``DummyCache`` to ``VALID_CACHE_BACKENDS``
- Updates the comparison with django-cache-machine & django-cacheops by
  checking features and measuring performance instead of relying on their
  documentations and a 2-years-ago experience of them

1.1.0
-----

**Backwards incompatible changes:**

- Adds Django 1.8 support and drops Django 1.6 & Python 2.6 support
- Merges the 3 API functions ``invalidate_all``, ``invalidate_tables``,
  & ``invalidate_models`` into a single ``invalidate`` function
  while optimising it

Other additions:

- Adds a ``get_last_invalidation`` function to the API and the equivalent
  template tag
- Adds a ``CACHALOT_ONLY_CACHABLE_TABLES`` setting in order to make a whitelist
  of the only table names django-cachalot can cache
- Caches queries with IP addresses, floats, or decimals in parameters
- Adds a Django check to ensure the project uses
  compatible cache and database backends
- Adds a lot of tests, especially to test django.contrib.postgres
- Adds a comparison with django-cache-machine and django-cacheops
  in the documentation

Fixed:

- Removes a useless extra invalidation during each write operation
  to the database, leading to a small speedup
  during data modification and tests
- The ``post_invalidation`` signal was triggered during transactions
  and was not triggered when using the API or raw write queries: both issues
  are now fixed
- Fixes a very unlikely invalidation issue occurring only when an error
  occurred in a transaction after a transaction of another database nested
  in the first transaction was committed, like this:

  .. code:: python

      from django.db import transaction

      assert list(YourModel.objects.using('another_db')) == []

      try:
          with transaction.atomic():
              with transaction.atomic('another_db'):
                  obj = YourModel.objects.using('another_db').create(name='test')
              raise ZeroDivisionError
      except ZeroDivisionError:
          pass

      # Before django-cachalot 1.1.0, this assert was failing.
      assert list(YourModel.objects.using('another_db')) == [obj]


1.0.3
-----

- Fixes an invalidation issue that could rarely occur when querying on a
  ``BinaryField`` with PostgreSQL, or with some geographic queries
  (there was a small chance that a same query with different parameters
  could erroneously give the same result as the previous one)
- Adds a ``CACHALOT_UNCACHABLE_TABLES`` setting
- Fixes a Django 1.7 migrations invalidation issue in tests
  (that was leading to this error half of the time:
  ``RuntimeError: Error creating new content types. Please make sure
  contenttypes is migrated before trying to migrate apps individually.``)
- Optimises tests when using django-cachalot
  by avoid several useless cache invalidations


1.0.2
-----

- Fixes an ``AttributeError`` occurring when excluding through a many-to-many
  relation on a child model (using multi-table inheritance)
- Stops caching queries with random subqueries – for example
  ``User.objects.filter(pk__in=User.objects.order_by('?'))``
- Optimises automatic invalidation
- Adds a note about clock synchronisation


1.0.1
-----

- Fixes an invalidation issue discovered by Helen Warren that was occurring
  when updating a ``ManyToManyField`` after executing using ``.exclude``
  on that relation. For example, ``Permission.objects.all().delete()`` was not
  invalidating ``User.objects.exclude(user_permissions=None)``
- Fixes a ``UnicodeDecodeError`` introduced with python-memcached 1.54
- Adds a ``post_invalidation`` signal


1.0.0
-----

Fixes a bug occurring when caching a SQL query using a non-ascii table name.


1.0.0rc
-------

Added:

- Adds an `invalidate_cachalot` command to invalidate django-cachalot
  from a script without having to clear the whole cache
- Adds the benchmark introduction, conditions & results to the documentation
- Adds a short guide on how to configure Redis as a LRU cache

Fixed:

- Fixes a rare invalidation issue occurring when updating a many-to-many table
  after executing a queryset generating a ``HAVING`` SQL statement –
  for example,
  ``User.objects.first().user_permissions.add(Permission.objects.first())``
  was not invalidating
  ``User.objects.annotate(n=Count('user_permissions')).filter(n__gte=1)``
- Fixes an even rarer invalidation issue occurring when updating a many-to-many
  table after executing a queryset filtering nested subqueries
  by another subquery through that many-to-many table – for example::

    User.objects.filter(
        pk__in=User.objects.filter(
            pk__in=User.objects.filter(
                user_permissions__in=Permission.objects.all())))
- Avoids setting useless cache keys by using table names instead of
  Django-generated table alias


0.9.0
-----

Added:

- Caches all queries implying ``Queryset.extra``
- Invalidates raw queries
- Adds a simple API containing:
  ``invalidate_tables``, ``invalidate_models``, ``invalidate_all``
- Adds file-based cache support for Django 1.7
- Adds a setting to choose if random queries must be cached
- Adds 2 settings to customize how cache keys are generated
- Adds a django-debug-toolbar panel
- Adds a benchmark

Fixed:

- Rewrites invalidation for a better speed & memory performance
- Fixes a stale cache issue occurring when an invalidation is done
  exactly during a SQL request on the invalidated table(s)
- Fixes a stale cache issue occurring after concurrent transactions
- Uses an infinite timeout

Removed:

- Simplifies ``cachalot_settings`` and forbids its use or modification


0.8.1
-----

- Fixes an issue with pip if Django is not yet installed


0.8.0
-----

- Adds multi-database support
- Adds invalidation when altering the DB schema using `migrate`, `syncdb`,
  `flush`, `loaddata` commands (also invalidates South, if you use it)
- Small optimizations & simplifications
- Adds several tests


0.7.0
-----

- Adds thread-safety
- Optimizes the amount of cache queries during transaction

0.6.0
-----

- Adds memcached support


0.5.0
-----

- Adds ``CACHALOT_ENABLED`` & ``CACHALOT_CACHE`` settings
- Allows settings to be dynamically overridden using ``cachalot_settings``
- Adds some missing tests

0.4.1
-----

- Fixes ``pip install``.

0.4.0 (**install broken**)
--------------------------

- Adds Travis CI and adds compatibility for:

  - Django 1.6 & 1.7
  - Python 2.6, 2.7, 3.2, 3.3, & 3.4
  - locmem & Redis
  - SQLite, PostgreSQL, MySQL

0.3.0
-----

- Handles transactions
- Adds lots of tests for complex cases

0.2.0
-----

- Adds a test suite
- Fixes invalidation for data creation/deletion
- Stops caching on queries defining ``select`` or ``where`` arguments
  with ``QuerySet.extra``

0.1.0
-----

Prototype simply caching all SQL queries reading the database
and trying to invalidate them when SQL queries modify the database.

Has issues invalidating deletions and creations.
Also caches ``QuerySet.extra`` queries but can’t reliably invalidate them.
No transaction support, no test, no multi-database support, etc.
