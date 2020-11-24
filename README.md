# pyreadstat_wheels3
Wheels for pyreadstat

This is a repo building wheels for pyreadstat using the multibuild package. 
It is a modification from the older repo .com/MacPython/pyreadstat-wheels.
The main modification respect to the old one is that files are uploaded to Anaconda Cloud 
instead of Rackspace (as the former was going to dissapear).

In order to do that a anaconda cloud account had to be set and the token retrived via WEB UI 
(also possible with CLI), set the Token in Travis and APPVEYOR as secret environment variable
for the project, and then just upload the wheels using the Anaconda-client in .travis.yml and
appveyor.yml. Interestingly in travis as we are using plain python and not anaconda, I had to 
install Anaconda-cli from github as the pypi version was too old and was causing errors. 

The wheels are then visible in https://anaconda.org/ofajardo/pyreadstat/files

Also, in appveyor we are building using MINGW with anaconda insetead of MVSC because readstat cannot be built
with MVSC at the moment.


########################################
Building and uploading pyreadstat wheels
########################################

We automate wheel building using this custom github repository that builds on
the travis-ci OSX machines, travis-ci Linux machines, and the Appveyor VMs.

The travis-ci interface for the builds is
https://travis-ci.org/MacPython/pyreadstat-wheels

Appveyor interface at
https://ci.appveyor.com/project/matthew-brett/pyreadstat-wheels

The driving github repository is
https://github.com/MacPython/pyreadstat-wheels

How it works
============

The wheel-building repository:

* does a fresh build of any required C / C++ libraries;
* builds a pyreadstat wheel, linking against these fresh builds;
* processes the wheel using delocate_ (OSX) or auditwheel_ ``repair``
  (Manylinux1_).  ``delocate`` and ``auditwheel`` copy the required dynamic
  libraries into the wheel and relinks the extension modules against the
  copied libraries;
* uploads the built wheels to a Rackspace container kindly donated by Rackspace
  to scikit-learn.

The resulting wheels are therefore self-contained and do not need any external
dynamic libraries apart from those provided as standard by OSX / Linux as
defined by the manylinux1 standard.

The ``.travis.yml`` file in this repository has a line containing the API key
for the Rackspace container encrypted with an RSA key that is unique to the
repository - see https://docs.travis-ci.com/user/encryption-keys.  This
encrypted key gives the travis build permission to upload to the Rackspace
containers we use to house the uploads.

Triggering a build
==================

You will likely want to edit the ``.travis.yml`` and ``appveyor.yml`` files to
specify the ``BUILD_COMMIT`` before triggering a build - see below.

You will need write permission to the github repository to trigger new builds
on the travis-ci interface.  Contact us on the mailing list if you need this.

You can trigger a build by:

* making a commit to the `pyreadstat-wheels` repository (e.g. with `git
  commit --allow-empty`); or
* clicking on the circular arrow icon towards the top right of the travis-ci
  page, to rerun the previous build.

In general, it is better to trigger a build with a commit, because this makes
a new set of build products and logs, keeping the old ones for reference.
Keeping the old build logs helps us keep track of previous problems and
successful builds.

Which pyreadstat commit does the repository build?
==================================================

The ``pyreadstat-wheels`` repository will build the commit specified in the
``BUILD_COMMIT`` at the top of the ``.travis.yml`` and ``appveyor.yml`` files.
This can be any naming of a commit, including branch name, tag name or commit
hash.

Note: when making a release, it's best to only push the commit (not the tag) of
the release to the ``pyreadstat`` repo, then change ``BUILD_COMMIT`` to the
commit hash, and only after all wheel builds completed successfully push the
release tag to the repo.  This avoids having to move or delete the tag in case
of an unexpected build/test issue.

Uploading the built wheels to pypi
==================================

* release container visible at
  https://3f23b170c54c2533c070-1c8a9b3114517dc5fe17b7c3f8c63a43.ssl.cf2.rackcdn.com

Be careful, these links point to containers on a distributed content delivery
network.  It can take up to 15 minutes for the new wheel file to get updated
into the containers at the links above.

When the wheels are updated, you can download them to your machine manually,
and then upload them manually to pypi, or by using twine_.  You can also use a
script for doing this, housed at :
https://github.com/MacPython/terryfy/blob/master/wheel-uploader
When the wheels are updated, you can download them to your machine manually,
and then upload them manually to pypi, or by using twine_.  You can also use a
script for doing this, housed at :
https://github.com/MacPython/terryfy/blob/master/wheel-uploader

For the ``wheel-uploader`` script, you'll need twine and `beautiful soup 4
<bs4>`_.

You will typically have a directory on your machine where you store wheels,
called a `wheelhouse`.   The typical call for `wheel-uploader` would then
be something like::

    VERSION=0.2.0
    CDN_URL=https://3f23b170c54c2533c070-1c8a9b3114517dc5fe17b7c3f8c63a43.ssl.cf2.rackcdn.com
    wheel-uploader -u $CDN_URL -s -v -w ~/wheelhouse -t all pyreadstat $VERSION

where:

* ``-u`` gives the URL from which to fetch the wheels, here the https address,
  for some extra security;
* ``-s`` causes twine to sign the wheels with your GPG key;
* ``-v`` means give verbose messages;
* ``-w ~/wheelhouse`` means download the wheels from to the local directory
  ``~/wheelhouse``.

``pyreadstat`` is the root name of the wheel(s) to download / upload, and
``0.2.0`` is the version to download / upload.

In order to upload the wheels, you will need something like this
in your ``~/.pypirc`` file::

    [distutils]
    index-servers =
        pypi

    [pypi]
    username:your_user_name
    password:your_password

So, in this case, `wheel-uploader` will download all wheels starting with
`pyreadstat-0.2.0-` from the URL in ``$CDN_URL`` above to ``~/wheelhouse``, then
upload them to PyPI.

Of course, you will need permissions to upload to PyPI, for this to work.

.. _manylinux1: https://www.python.org/dev/peps/pep-0513
.. _twine: https://pypi.python.org/pypi/twine
.. _bs4: https://pypi.python.org/pypi/beautifulsoup4
.. _delocate: https://pypi.python.org/pypi/delocate
.. _auditwheel: https://pypi.python.org/pypi/auditwheel
