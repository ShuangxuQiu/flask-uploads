=============
Flask-Uploads
=============

Flask-Uploads allows your application to flexibly and efficiently handle file
uploading and serving the uploaded files. You can create different sets of
uploads - one for document attachments, one for photos, etc. - and the
application can be configured to save them all in different places and to
generate different URLs for them.

Fully Python 3-compatible.

There's an example application in ``example/`` that requires
Flask-CouchDB_.

.. _Flask-CouchDB: https://pythonhosted.org/Flask-CouchDB/


Tests
=====

To run the tests:

.. code:: sh

  $ pip install nose
  $ nosetests


| Currently maintained by Jeff Widman
| Originally written by Matthew "LeafStorm" Frazier
