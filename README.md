[![Build Status](https://travis-ci.org/kpn-digital/django-etcd-settings.svg?branch=master)](https://travis-ci.org/kpn-digital/django-etcd-settings)

# django-etcd-settings

## Goal

This application allows you to extend the Django settings as configured in the
`settings.py` file with:

* Environment dependent values
* Values in different config sets, identified by name, which can be selected on
  a 'per request' basis using the `X-DYNAMIC-SETTINGS` HTTP header

## Installation

For now, just install it by pointing to this repository. You can either:

1. Clone this repository and run `make test` to check the library out and
   contribute to it

2. Have your Django application depend on this by using something like this in
   your `requirements.txt` file:
   `https://github.com/kpn-digital/django-etcd-settings.git/kpn-digital/django-etcd-settings.git@master#egg=etcd_settings`


## Compatibility

This application has been tested with Python 2.7 and Django 1.7

## Configuration

This Django application uses the following configuration keys:

* `DJES_ETCD_DETAILS`: a dict with 'host', 'port', 'protocol', 'prefix',
    'long_polling_timeout' and 'long_polling_safety_delay' (both in seconds).
    'prefix' is a string to be used as base path for all configuration
    managed by this app.
    i.e. '/config/api' will result in '/config/api/<ENV>' and
    '/config/api/extensions/' to be used for environment defaults and
    config_sets respectively
    Timeouts default to 50 and 5 seconds respectively.
    If `DJES_ETCD_SETTINGS` is None, this app will start with no errors and
    etcd_settings.settings will resolve to django.conf.settings plus your
    DJES_DEV_PARAMS overwrites
    i.e.
    ```
    ETCD_DETAILS = dict(
        host='localhost', port=4000, protocol='http',
        long_polling_timout=50, long_polling_safety_delay=5
    )
    ```

* `DJES_DEV_PARAMS`: A module with local overwrites, generally used for
    development. The overwrites must be capitalized module attributes.
    These overwrites will have preference over development settings on ETCD,
    but not over configset overwrites indicated by the `X-DYNAMIC-SETTINGS`
    HTTP header

* `DJES_ENV`: A string with the name of the environment in which the code is
    running. This will be used for accessing the env_defaults in
    ETCD in a directory with that name
    i.e. 'test', 'staging', 'prod'...

* `DJES_REQUEST_GETTER`: path to a function which accesses the HTTP request
    object being handled. Ensuring access to this value can be implemented
    with, for instance, middleware. This settings is only used to allow
    config overwrites on runtime based on predifined config_sets. In case you
    don't want to use this functionality, just set this setting to None
    i.e. 'middleware.thread_local.get_current_request'

* `DJES_WSGI_FILE`: path to the `wsgi.py` file for the django
    project. If not None, the monitoring of environment configuration will
    perform a `touch` of the file everytime the env_defaults are updated, so
    that all processes consuming settings from `django.conf` can consume the
    latest settings available as well
    The path can be absolute or relative to the 'manage.py' file.
    i.e. /project/src/wsgi.py, wsgi.py

Then, add `etcd_settings` to the list of `INSTALLED_APPS` before any other that
requires dynamic settings.

From your code, just do `from etcd_settings import settings` instead of `from
django.conf import settings`.

In case you want to use `etcd_settings` to modify some values in your standard
Django settings.py file (i.e. Database config), you can use the following
snippet in your settings file, as high as possible in the file and immediately
under the `DJES_*` settings definition:

```
import etcd_settings.loader
extra_settings = etcd_settings.loader.get_overwrites(
    DJES_ENV, DJES_DEV_PARAMS, DJES_ETCD_DETAILS)
locals().update(extra_settings)
```

## TODO

* Extending the unit tests in order to cover `etcd_settings.proxy` and more of
  `etcd_settings.manager`

* Moving this repository to [Github](http://www.github.com) and updating `url`
  and `download_url` in `setup.py` file with the new repo location

* Uploading this package to [PyPi](https://pypi.python.org). A user needs to be
  configured in `.pypirc`. After that, run:
  ```
  UPLOAD_TARGET=<upload_target> make upload
  ```
  ... where upload_target is one of `pypitest` or `pypi`
