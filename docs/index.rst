=============
Flask-Uploads
=============
.. currentmodule:: flask_uploads

Flask-Uploads allows your application to flexibly and efficiently handle file
uploading and serving the uploaded files.
You can create different sets of uploads - one for document attachments, one
for photos, etc. - and the application can be configured to save them all in
different places and to generate different URLs for them.

.. contents::
   :local:
   :backlinks: none

Upload Sets
===========
Applications define one or more "upload sets," which have names like photos,
music, and attachments. An "upload set" is a single collection of files that are
managed independently. For each upload set you can:

* Define the list of file extensions that the upload set will allow/deny.
* Define the locations on the filesystem where uploads to each set are stored.
* Define the public URL of the files in the upload set. This is useful for using
  a Web server like Apache or Nginx to directly serve the uploaded files, which
  is much faster than serving them through Flask.

To create a new upload set, just declare it in the code::

   photos = UploadSet('photos', IMAGES)

And then you can use the `~UploadSet.save` method to save uploaded files and
`~UploadSet.path` and `~UploadSet.url` to access them. For example::

   @app.route('/upload', methods=['GET', 'POST'])
   def upload():
       if request.method == 'POST' and 'photo' in request.files:
           filename = photos.save(request.files['photo'])
           # save photo metadata to your db, for example with Flask-SQLAlchemy:
           photo = Photo(filename=filename, user=g.user.id)
           db.session.add(photo)
           db.session.commit(photo)
           flash("Photo saved.")
           return redirect(url_for('show', id=photo.id))
       return render_template('upload.html')

   @app.route('/photo/<id>')
   def show(id):
       # load photo metadata from your db, for example with Flask-SQLAlchemy:
       photo = Photo.query.get(id)
       if photo is None:
           abort(404)
       url = photos.url(photo.filename)
       return render_template('show.html', url=url, photo=photo)

If you have a "default location" for storing uploads - for example, if your
app has an "instance" directory like `Zine`_ and uploads should be saved to
the instance directory's ``uploads`` folder - you can pass a ``default_dest``
callable to the set constructor. It takes the application as its argument.
For example::

   media = UploadSet('media', default_dest=lambda app: app.instance_path)

This won't prevent a different destination from being set in the config,
though. It's just to save your users a little configuration time.

.. _Zine: http://zine.pocoo.org/


App Configuration
=================
An upload set's configuration is stored on an app. That way, you can have
upload sets being used by multiple apps at once. You use the
`configure_uploads` function to load the configuration for the upload sets.
You pass in the app and all of the upload sets you want configured. Calling
`configure_uploads` more than once is safe. ::

    configure_uploads(app, (photos, media))

If your app has a factory function, that is a good place to call this
function.

By default, Flask `doesn't put any limits on the size of the uploaded data
<http://flask.pocoo.org/docs/0.10/patterns/fileuploads/#improving-uploads>`_. To
protect your application, you should set `MAX_CONTENT_LENGTH` in your
application config to cap the maximum upload size.


Configuration
=============
You can customize the settings of different upload sets by adjusting the
application's configuration settings. Even if you're just deploying an
application that uses Flask-Uploads, you can still use these settings to
override the settings specified by the application author.

The settings below apply for a single set of uploads, replacing `FILES` with
the name of the set (i.e. `PHOTOS`, `ATTACHMENTS`):

`UPLOADED_FILES_DEST`
    This indicates the directory uploaded files will be saved to.

`UPLOADED_FILES_URL`
    If you have a server set up to serve the files in this set, this should be
    the URL they are publicly accessible from. Include the trailing slash.

`UPLOADED_FILES_ALLOW`
    This lets you allow file extensions not allowed by the upload set in the
    code.

`UPLOADED_FILES_DENY`
    This lets you deny file extensions allowed by the upload set in the code.

To save on configuration time, there are two settings you can provide
that apply as "defaults" if you don't provide the proper settings otherwise.

`UPLOADS_DEFAULT_DEST`
    If you set this, then if an upload set's destination isn't otherwise
    declared, then its uploads will be stored in a subdirectory of this
    directory. For example, if you set this to ``/var/uploads``, then a set
    named photos will store its uploads in ``/var/uploads/photos``.

`UPLOADS_DEFAULT_URL`
    If you have a server set up to serve from `UPLOADS_DEFAULT_DEST`, then
    set the server's base URL here. Continuing the example above, if
    ``/var/uploads`` is accessible from ``http://localhost:5001``, then you
    would set this to ``http://localhost:5001/`` and URLs for the photos set
    would start with ``http://localhost:5001/photos``. Include the trailing
    slash.

However, you don't have to set any of the ``_URL`` settings - if you don't,
then they will be served internally by Flask. They are just there so if you
have heavy upload traffic, you can have a faster production server like Nginx
or Lighttpd serve the uploads.

You should also set `MAX_CONTENT_LENGTH` to limit the size of uploaded files.


File Upload Forms
=================
To actually upload the files, you need to properly set up your form. A form
that uploads files needs to have its method set to POST and its enctype
set to ``multipart/form-data``. If it's set to GET, it won't work at all, and
if you don't set the enctype, only the filename will be transferred.

The field itself should be an ``<input type=file>``.

.. code-block:: html+jinja

    <form method=POST enctype=multipart/form-data action="{{ url_for('upload') }}">
        ...
        <input type=file name=photo>
        ...
    </form>


API Documentation
=================
This documentation is generated directly from the source code.


Upload Sets
-----------
.. autoclass:: UploadSet
   :members:

.. autoclass:: UploadConfiguration


Application Setup
-----------------
.. autofunction:: configure_uploads


Extension Constants
-------------------
These are some default sets of extensions you can pass to the `UploadSet`
constructor.

.. autoclass:: AllExcept

.. autodata:: DEFAULTS

.. autodata:: ALL

.. autodata:: TEXT

.. autodata:: IMAGES

.. autodata:: AUDIO

.. autodata:: DOCUMENTS

.. autodata:: DATA

.. autodata:: SCRIPTS

.. autodata:: ARCHIVES

.. autodata:: EXECUTABLES


Testing Utilities
-----------------
.. autoclass:: TestingFileStorage


Backwards Compatibility
=======================
Version 0.2.0
-------------
* The `patch_request_class` function was removed as it's unnecessary from Flask
  0.6 onward.
* Filenames without extensions are no longer lowercased by `lowercase_ext`,
  only the extension is returned in lowercase, if an extension exists.

Version 0.1.3
-------------
* The `_uploads` module/blueprint will not be registered if it is not needed
  to serve uploads.


Version 0.1.1
-------------
* `patch_request_class` now changes `max_content_length` instead of
  `max_form_memory_size`.
