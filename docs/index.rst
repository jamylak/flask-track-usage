Flask-Track-Usage |release|
===========================

Basic metrics tracking for your `Flask`_ application. This focuses more on ip addresses/locations rather than tracking specific users pathing through an application. No extra cookies or javascript is used for usage tracking.

* Simple. It's a Flask extension.
* Supports either include or exempt for views.
* Provides lite abstraction for data retrieval.
* Optional `freegeoip.net <http://freegeoip.net/>`_ integration.
* Multiple storage options.


.. _Flask: http://flask.pocoo.org/


Installation
------------

.. warning::
   This release is not 100% backwards compatible with the 0.0.x series of releases.

Requirements
~~~~~~~~~~~~
* Flask: http://flask.pocoo.org/
* A storage object to save the metrics data with

Via pip
~~~~~~~
::

   $ pip install Flask-Track-Usage


Via source
~~~~~~~~~~
::

   $ python setup.py install

Usage
-----

::

    # Create the Flask 'app'
    from flask import Flask
    app = Flask(__name__)

    # Set the configuration items manually for the example
    app.config['TRACK_USAGE_USE_FREEGEOIP'] = False
    app.config['TRACK_USAGE_INCLUDE_OR_EXCLUDE_VIEWS'] = 'include'

    # We will just print out the data for the example
    from flask.ext.track_usage import TrackUsage
    from flask.ext.track_usage.storage.printer import PrintStorage

    # Make an instance of the extension
    t = TrackUsage(app, PrintStorage())

    # Make an instance of the extension
    t = TrackUsage(app, storage)

    # Include the view in the metrics
    @t.include
    @app.route('/')
    def index():
        return "Hello"

    # Run the application!
    app.run(debug=True)


Blueprint Support
-----------------
Blueprints can be included or excluded from Flask-TrackUsage in their entirety.

::

    # ...
    app.config['TRACK_USAGE_INCLUDE_OR_EXCLUDE_VIEWS'] = 'include'

    # Make an instance of the extension
    t = TrackUsage(app, PrintStorage())

    from my_blueprints import a_bluprint

    # Now ALL of a_blueprint's views will be in the include list
    t.include_bleprint(a_blueprint)


::

    # ...
    app.config['TRACK_USAGE_INCLUDE_OR_EXCLUDE_VIEWS'] = 'exclude'

    # Make an instance of the extension
    t = TrackUsage(app, PrintStorage())

    from my_blueprints import a_bluprint

    # Now ALL of different_blueprints will be in the exclude list
    t.exclude_blueprint(a_blueprint)


Configuration
-------------

TRACK_USAGE_USE_FREEGEOIP
~~~~~~~~~~~~~~~~~~~~~~~~~
**Values**: True, False

**Default**: False

Turn FreeGeoIP integration on or off

TRACK_USAGE_INCLUDE_OR_EXCLUDE_VIEWS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
**Values**: include, exclude

**Default**: exclude

If views should be included or excluded by default.

* When set to *exclude* each routed view must be explicitly included via decorator or blueprint include method. If a routed view is not included it will not be tracked.
* When set to *include* each routed view must be explicitly excluded via decorator or blueprint exclude method. If a routed view is not excluded it will be tracked.

Storage
-------
The following are built in, ready to use storage backends.

.. note:: Inputs for set_up should be passed in __init__ when creating a storage instance

printer.PrintStorage
~~~~~~~~~~~~~~~~~~~~
.. autoclass:: flask_track_usage.storage.printer.PrintStorage


mongo.MongoPiggybackStorage
~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. autoclass:: flask_track_usage.storage.mongo.MongoPiggybackStorage
    :members:


mongo.MongoStorage
~~~~~~~~~~~~~~~~~~
.. autoclass:: flask_track_usage.storage.mongo.MongoStorage
    :members:


sql.SQLStorage
~~~~~~~~~~~~~~~~~~
.. autoclass:: flask_track_usage.storage.sql.SQLStorage
    :members:


Retrieving Data
---------------
All storage backends, other than printer.PrintStorage, provide get_usage.

.. autoclass:: flask_track_usage.storage.Storage
    :members: get_usage

Results that are returned from all instances of get_usage should **always** look like this:

.. code-block:: python

 [
     {
             'url': str,
             'user_agent': {
                 'browser': str,
                 'language': str,
                 'platform': str,
                 'version': str,
             },
             'blueprint': str,
             'view_args': dict or None
             'status': int,
             'remote_addr': str,
             'authorization': bool
             'ip_info': str or None,
             'path': str,
             'speed': float,
             'date': datetime,
     },
     {
         ....
     }
 ]
