==============
Important Note
==============

This is a fork of easy-thumbnails v1.4 and it has fixes for remote storage(like amazon S3) performance issues like querying to db for each
thumbnail or checking its existency and modified time on remote storage. It simply caches modified time of your 
thumbnail with your default cache backend.

It supposed to work fine for local storages too. It includes three more settings than original easy-thumbnails package.

``FILE_REMOTE_STORAGE = False`` default is false but if you are using remote storages for your ``DEFAULT_FILE_STORAGE``, 
you definitely need to set it to True. Otherwise, it will still solve the "querying to db" problem but keep requesting
the modified time to remote storage instead of caching it on your cache backend.

``THUMBNAIL_REMOTE_STORAGE = False`` default is false but if you are using remote storages for your 
``THUMBNAIL_DEFAULT_STORAGE``, you definitely need to set it to True. Otherwise, it will still solve the 
"querying to db" problem but keep requesting the modified time to remote storage instead of caching it on your 
cache backend.

``EASY_CACHE_TIMEOUT = 60 * 60 * 24 * 30`` default is 30 days for caching ``modified_time`` of source image or 
thumbnail image.

``Invalidation``:
There are few cases for invalidations. For example, if your image is replaced with a different named image, then you don't 
need to invalidate cache necessarily. However, if your image is replaced with same name but different quality, then
cache needs to be invalidated definitely. For this purpose, there is a function called ``invalidate_easy_cache`` 
under ``utils.py``. It consumes only source image name.



===============
Easy Thumbnails
===============

.. image:: https://secure.travis-ci.org/SmileyChris/easy-thumbnails.png?branch=master
    :alt: Build Status
    :target: http://travis-ci.org/SmileyChris/easy-thumbnails


A powerful, yet easy to implement thumbnailing application for Django 1.4+

Below is a quick summary of usage. For more comprehensive information, view the
`full documentation`__ online or the peruse the project's ``docs`` directory.

__ http://easy-thumbnails.readthedocs.org/en/latest/index.html


Installation
============

Run ``pip install easy-thumbnails``, or for the `in-development version`__
run ``pip install easy-thumbnails==dev``.

__ https://github.com/SmileyChris/easy-thumbnails/tarball/master#egg=easy_thumbnails-dev

Add ``easy_thumbnails`` to your ``INSTALLED_APPS`` setting::

    INSTALLED_APPS = (
        ...
        'easy_thumbnails',
    )

If you have South installed then run ``manage.py migrate easy_thumbnails``,
otherwise just run ``manage.py syncdb``.


Example usage
=============

Thumbnail options can be predefined in ``settings.THUMBNAIL_ALIASES`` or just
specified in the template or Python code when run.

Using a predefined alias
------------------------

Given the following setting::

    THUMBNAIL_ALIASES = {
        '': {
            'avatar': {'size': (50, 50), 'crop': True},
        },
    }

Template::

    {% load thumbnail %}
    <img src="{{ profile.photo|thumbnail_url:'avatar' }}" alt="" />

Python::

    from easy_thumbnails.files import get_thumbnailer
    thumb_url = get_thumbnailer(profile.photo)['avatar'].url

Manually specifying size / options
----------------------------------

Template::

    {% load thumbnail %}
    <img src="{% thumbnail profile.photo 50x50 crop %}" alt="" />

Python::

    from easy_thumbnails.files import get_thumbnailer
    options = {'size': (100, 100), 'crop': True}
    thumb_url = get_thumbnailer(profile.photo).get_thumbnail(options).url


Fields
======

You can use ``ThumbnailerImageField`` (or ``ThumbnailerFileField``) for easier
access to retrieve or generate thumbnail images.

For example::

    from easy_thumbnails.fields import ThumbnailerImageField

    class Profile(models.Model):
        user = models.OneToOneField('auth.User')
        photo = ThumbnailerImageField(upload_to='photos', blank=True)

Accessing the field's predefined alias in a template::

    {% load thumbnail %}
    <img src="{{ profile.photo.avatar.url }}" alt="" />

Accessing the field's predefined alias in Python code::

    thumb_url = profile.photo['avatar'].url


Thumbnail options
=================

``crop``
--------

Before scaling the image down to fit within the ``size`` bounds, it first cuts
the edges of the image to match the requested aspect ratio.

Use ``crop="smart"`` to try to keep the most interesting part of the image,

Use ``crop="0,10"`` to crop from the left edge and a 10% offset from the
top edge. Crop from a single edge by leaving dimension empty (e.g.
``crop=",0"``). Offset from the right / bottom by using negative numbers
(e.g., crop="-0,-10").

Often used with the ``upscale`` option, which will allow enlarging of the image
during scaling.

``quality=XX``
--------------

Changes the quality of the output JPEG thumbnail. Defaults to ``85``.

In Python code, this is given as a separate option to the ``get_thumbnail``
method rather than just alter the other

Other options
-------------

Valid thumbnail options are determined by the "thumbnail processors" installed.

See the `reference documentation`__ for a complete list of options provided by
the default thumbnail processors.

__ http://easy-thumbnails.readthedocs.org/en/latest/ref/processors/
