================
django-db-logger
================

.. image:: https://travis-ci.org/CiCiUi/django-db-logger.svg?branch=master
    :target: https://travis-ci.org/CiCiUi/django-db-logger

Django logging in database.
For large projects please use `Sentry <https://github.com/getsentry/sentry>`_

Screenshot
----------
.. image:: https://ciciui.github.io/django-db-logger/static/img/django-db-logger.png
    :target: https://travis-ci.org/CiCiUi/django-db-logger

Dependency
----------
* Django>=3.2
* Python 3.8+

License
-------
WTFPL

Quick start
-----------

1. Install

.. code-block:: bash

    pip install django-db-logger

2. Add "django_db_logger" to your ``INSTALLED_APPS`` setting like this

.. code-block:: python

    INSTALLED_APPS = (
        ...
        'django_db_logger',
    )

3. Add handler and logger to ``LOGGING`` setting like this

.. code-block:: python

    LOGGING = {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'verbose': {
                'format': '%(levelname)s %(asctime)s %(module)s %(process)d %(thread)d %(message)s'
            },
            'simple': {
                'format': '%(levelname)s %(asctime)s %(message)s'
            },
        },
        'handlers': {
            'db_log': {
                'level': 'DEBUG',
                'class': 'django_db_logger.db_log_handler.DatabaseLogHandler'
            },
        },
        'loggers': {
            'db': {
                'handlers': ['db_log'],
                'level': 'DEBUG'
            },
            'django.request': { # logging 500 errors to database
                'handlers': ['db_log'],
                'level': 'ERROR',
                'propagate': False,
            }
        }
    }

4. Run ``python manage.py migrate`` to create django-db-logger models.
5. Use ``django-db-logger`` like this

.. code-block:: python

    import logging
    db_logger = logging.getLogger('db')

    db_logger.info('info message')
    db_logger.warning('warning message')

    try:
        1/0
    except Exception as e:
        db_logger.exception(e)



Options
-------
1. DJANGO_DB_LOGGER_ADMIN_LIST_PER_PAGE: integer. list per page in admin view. default ``10``
2. DJANGO_DB_LOGGER_ENABLE_FORMATTER: boolean. Using ``formatter`` options to format message. ``True`` or ``False``, default ``False``


Define your own behaviour to log additional details:
----------------------------------------------------
Extend the ``DatabaseLogHandler`` to add your own configurations and functionality to  the ``add_details`` method:

.. code-block:: python

    # my_log_handlers.py
    from django_db_logger.db_log_handler import DatabaseLogHandler

    class MyDatabaseLogHandler(DatabaseLogHandler):
        def add_details(self, record, log_fields):
            """
            Extract relevant details from the record's request:
                * request path
                * used HTTP method
                * used GET params (within URL)
            """
            request = getattr(record, 'request', None)
            if request:
                log_fields['details'] = {
                    'path': getattr(request, 'path', None),
                    'method': getattr(request, 'method', None),
                    'params': getattr(request, 'GET', None),
                }


And make sure to use your own class in the logging configuration:
.. code-block:: python

    # settings.py
    LOGGING["handlers"]["db_log"] = {
        "class": "my_project.my_log_handlers.MyDatabaseLogHandler",
        "level": "DEBUG",
    }


Build your own database logger :hammer:
---------------------------------------
1. Create a new app and add it to ``INSTALLED_APPS``
2. Copy files ``django-db-logger/models.py``, ``django-db-logger/admin.py``, ``django-db-logger/db_log_handler.py`` to the app folder
3. Replace ``DJANGO_DB_LOGGER_ADMIN_LIST_PER_PAGE`` in ``admin.py`` with an integer
4. Replace ``DJANGO_DB_LOGGER_ENABLE_FORMATTER`` in `db_log_handler.py` with ``True`` or ``False``. Remove ``MSG_STYLE_SIMPLE``, it was not used.
5. Replace logger class ``django_db_logger.db_log_handler.DatabaseLogHandler`` in your Settings with the new logger class
6. Customize the logger to meet your needs. :beer:
