.. raw-sql-migrate documentation master file, created by
   sphinx-quickstart on Tue Jun 02 23:08:26 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


Goal
====
Raw-sql-migrate is tool for managing your raw SQL migrations.


Config
======
In order to use migrate api use should make an instance of Config class found in raw_sql_migrate package.
By default one config instance is created in init module which tries to load options data
from raw_sql_migrate.yaml in your project root. Still you can create config based on yaml file located in
another directory.

.. code-block:: python

    from raw_sql_migrate import Config
    config = Config().init_from_file(path_to_file)


Or you can just pass all params to constructor:

.. code-block:: python

    from raw_sql_migrate import Config
    config = Config(database, history_table_name, packages)

where params are mappings to yaml config structure:

.. code-block:: yaml

    database:
        engine: engine backend module
        host: database host
        port: database port
        name: database name
        user: user name
        password: user password
        additional_driver_param1: value
        additional_driver_param2: value
    history_table_name: migration history table name
    packages:
        - package_a
        - package_b
        - package_c.package_d

where available options are:

* raw_sql_migrate.engines.postgresql_psycopg2 (requires psycopg2 package)
* raw_sql_migrate.engines.mysql (requires MySQLdb-python package)

Also you can pass specific param to drivers connect method, just add this param in database section.
Packages param is a list of packages where to search for new migration for api.forward command.

Api
===
In order to create and run migrations you need to make instance of Api class from
raw_sql_migrate.api module and pass config instance if needed. Otherwise default config
will be used.


Creating new migration
----------------------
In order to create new migration just call create method:

.. code-block:: python

    api.create(package='package_a.package_b', name='initial')

Calling it will create new migrations history table, migrations directory
in the package and py migration file/

Migrating forward
-----------------
In order to migrate forward call

.. code-block:: python

    api.forward(package, migration_number=42)

Note: to migrate all not applied migrations you should skip migration_number param.

Migrating backward
------------------
In order to migrate forward call

.. code-block:: python

    api.backward(package, migration_number=1)

Note: to migrate to initial state you should pass migration_number as 1.

Migrations status
-----------------
To get latest migration data for tracked packages call method:

.. code-block:: python

    api.status(package='package_a.package_b')

In order to get info for all tracked packages just omit package attribute.
The result of this method is python dictionary:

.. code-block:: python

    {
        'package name': {
            'name': 'migration name',
            'processed_at': datetime when migration was applied,
        },
    }

Squashing migrations
--------------------
Sometimes there can be situation when you want to merge your dev migrations before
publishing them to repository. In this case use squash method:

.. code-block:: python

    api.squash(
        package='package_a.package_b',
        begin_from=42,
        name='squashed_migration'
    )

This example does next things:
It searches for not applied migration in package begining from number 42, reads their
content and appends it to result forward and backward functions. After all migrations
were processed command writes new migration file with 'begin_from' number and renames
squashed migrations with '_squashed' prefix. Note that command can't squash already
applied migrations.