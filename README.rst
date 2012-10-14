.. -*- mode: rst; coding: utf-8 -*-

==============================================================================
pypiserver4sae - SAE based minimal PyPI server for use with pip/easy_install
==============================================================================



.. contents:: Table of Contents
  :backlinks: top


pypiserver is a minimal PyPI compatible server. It can be used to
serve a set of packages and eggs to easy_install or pip.

This fork of pypiserver is for Sina App Engine platform enviroment only, and use wsgi as interface. 

You are able to setup a productive personal PyPI index in 5 mins.

For more options, please check https://github.com/schmir/pypiserver .

pypiserver 是最简化的PyPI 兼容服务器。它可以用来提供easy_install 或 pip 使用的packages 和 eggs服务。

此分支版本专门为SAE（Sina App Engine）定制，可以通过wsgi接口运行在模拟环境和SAE上。

你可以在5分钟内搭建生产级的个人PyPI索引服务。

本地服务及更多其他方案请参考https://github.com/schmir/pypiserver .

Installation on SAE
====================
使用SAE storage 服务作为存储方案，使用之前请启用 storage服务。

pypiserver4sae on SAE using wsgi interface::

  import os, sys
  
  sys.path.append(os.path.join(os.path.dirname(__file__), 'pypiserver').replace('\\','/'))
  
  import sae
  from pypiserver import app
  
  domain = "main"   #replace the storage domain name
  
  application = sae.create_wsgi_app(app(root=domain,
          redirect_to_fallback=True,
          fallback_url="http://pypi.python.org/simple"))


Test on SAE
===========
Test my demo server on http://pypiserver.sinaapp.com/simple/ ::

  $ easy_install -i http://pypiserver.sinaapp.com/simple/ an_example_pypi_project
  Searching for an-example-pypi-project
  Reading http://pypiserver.sinaapp.com/simple/an_example_pypi_project/
  Best match: an-example-pypi-project 0.0.5
  Downloading http://pypiserver.sinaapp.com/packages/an_example_pypi_project-0.0.5.zip
  Processing an_example_pypi_project-0.0.5.zip
  Running an_example_pypi_project-0.0.5/setup.py -q bdist_egg --dist-dir /tmp/easy_install-9cPbRY/an_example_pypi_project-0.0.5/egg-dist-tmp-19gZKR
  zip_safe flag not set; analyzing archive contents...
  Adding an-example-pypi-project 0.0.5 to easy-install.pth file
  
  Installed /home/felix/lab/SAEENV/lib/python2.7/site-packages/an_example_pypi_project-0.0.5-py2.7.egg
  Processing dependencies for an-example-pypi-project
  Finished processing dependencies for an-example-pypi-project


Alternative Installation as standalone script
=============================================
For more options, please check https://github.com/schmir/pypiserver .

Detailed Usage
=================================
pypi-server -h will print a detailed usage message::

  pypi-server [OPTIONS] [PACKAGES_DIRECTORY]
    start PyPI compatible package server serving packages from
    PACKAGES_DIRECTORY. If PACKAGES_DIRECTORY is not given on the
    command line, it uses the default ~/packages.  pypiserver scans this
    directory recursively for packages. It skips packages and
    directories starting with a dot.

  pypi-server understands the following options:

    -p PORT, --port PORT
      listen on port PORT (default: 8080)

    -i INTERFACE, --interface INTERFACE
      listen on interface INTERFACE (default: 0.0.0.0, any interface)

    -P PASSWORD_FILE, --passwords PASSWORD_FILE
      use apache htpasswd file PASSWORD_FILE in order to enable password
      protected uploads.

    --disable-fallback
      disable redirect to real PyPI index for packages not found in the
      local index

    --fallback-url FALLBACK_URL
      for packages not found in the local index, this URL will be used to
      redirect to (default: http://pypi.python.org/simple)

    --server METHOD
      use METHOD to run the server. Valid values include paste,
      cherrypy, twisted, gunicorn, gevent, wsgiref, auto. The
      default is to use "auto" which chooses one of paste, cherrypy,
      twisted or wsgiref.

    -r PACKAGES_DIRECTORY, --root PACKAGES_DIRECTORY
      [deprecated] serve packages from PACKAGES_DIRECTORY

  pypi-server -h
  pypi-server --help
    show this help message

  pypi-server --version
    show pypi-server's version

  pypi-server -U [OPTIONS] [PACKAGES_DIRECTORY]
    update packages in PACKAGES_DIRECTORY. This command searches
    pypi.python.org for updates and shows a pip command line which
    updates the package.

  The following additional options can be specified with -U:

    -x
      execute the pip commands instead of only showing them

    -d DOWNLOAD_DIRECTORY
      download package updates to this directory. The default is to use
      the directory which contains the latest version of the package to
      be updated.

    -u
      allow updating to unstable version (alpha, beta, rc, dev versions)

  Visit http://pypi.python.org/pypi/pypiserver for more information.



Configuring pip/easy_install
============================
Always specifying the the pypi url on the command line is a bit
cumbersome. Since pypi-server redirects pip/easy_install to the
pypi.python.org index if it doesn't have a requested package, it's a
good idea to configure them to always use your local pypi index.

pip
-----
For pip this can be done by setting the environment variable
PIP_INDEX_URL in your .bashrc/.profile/.zshrc::

  export PIP_INDEX_URL=http://localhost:8080/simple/

or by adding the following lines to ~/.pip/pip.conf::

  [global]
  index-url = http://localhost:8080/simple/

easy_install
------------
For easy_install it can be configured with the following setting in
~/.pydistutils.cfg::

  [easy_install]
  index_url = http://localhost:8080/simple/

Managing the package directory
==============================
pypi-server's -U option makes it possible to search for updates of
available packages. It scans the package directory for available
packages and searches on pypi.python.org for updates. Without further
options 'pypi-server -U' will just print a list of commands which must
be run in order to get the latest version of each package. Output
looks like::

  checking 106 packages for newer version

  .........u.e...........e..u.............
  .....e..............................e...
  ..........................

  no releases found on pypi for PyXML, Pymacs, mercurial, setuptools

  # update raven from 1.4.3 to 1.4.4
  pip -q install --no-deps -i http://pypi.python.org/simple -d /home/ralf/packages/mirror raven==1.4.4

  # update greenlet from 0.3.3 to 0.3.4
  pip -q install --no-deps -i http://pypi.python.org/simple -d /home/ralf/packages/mirror greenlet==0.3.4

It first prints for each package a single character after checking the
available versions on pypi. A dot means the package is up-to-date, 'u'
means the package can be updated and 'e' means the list of releases on
pypi is empty. After that it show a pip command line which can be used
to update a one package. Either copy and paste that or run
"pypi-server -Ux" in order to really execute those commands. You need
to have pip installed for that to work however.

Specifying an additional '-u' option will also allow alpha, beta and
release candidates to be downloaded. Without this option these
releases won't be considered.


Optional dependencies
=====================
- pypiserver ships with it's own copy of bottle. It's possible to use
  bottle with different WSGI servers. pypiserver chooses any of the
  following paste, cherrypy, twisted, wsgiref (part of python) if
  available.
- pypiserver relies on the passlib module for parsing apache htpasswd
  files. You need to install it, when using the -P, --passwords
  option. The following command will do that::

    pip install passlib


Using a different WSGI server
=============================
For more options, please check https://github.com/schmir/pypiserver .


Bugs
=============
pypiserver does not implement the full API as seen on PyPI_. It
implements just enough to make easy_install and pip install work.

The following limitations are known:

- pypiserver doesn't implement the XMLRPC interface: pip search
  will not work.
- pypiserver doesn't implement the json based '/pypi' interface. pyg_
  uses that and will not work.

Please use github's bugtracker
https://github.com/schmir/pypiserver/issues if you find any other
bugs.


License
=============
pypiserver contains a copy of bottle_ which is available under the
MIT license::

  Copyright (c) 2010, Marcel Hellkamp.

  Permission is hereby granted, free of charge, to any person obtaining a copy
  of this software and associated documentation files (the "Software"), to deal
  in the Software without restriction, including without limitation the rights
  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
  copies of the Software, and to permit persons to whom the Software is
  furnished to do so, subject to the following conditions:

  The above copyright notice and this permission notice shall be included in all
  copies or substantial portions of the Software.

  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
  SOFTWARE.


The remaining part is distributed under the zlib/libpng license::

  Copyright (c) 2011 Ralf Schmitt

  This software is provided 'as-is', without any express or implied
  warranty. In no event will the authors be held liable for any damages
  arising from the use of this software.

  Permission is granted to anyone to use this software for any purpose,
  including commercial applications, and to alter it and redistribute it
  freely, subject to the following restrictions:

  1. The origin of this software must not be misrepresented; you must not
     claim that you wrote the original software. If you use this software
     in a product, an acknowledgment in the product documentation would be
     appreciated but is not required.

  2. Altered source versions must be plainly marked as such, and must not be
     misrepresented as being the original software.

  3. This notice may not be removed or altered from any source
     distribution.


Similar Projects
====================
There are lots of other projects, which allow you to run your own
PyPI server. If pypiserver doesn't work for you, try one of the
following alternatives:

chishop (http://pypi.python.org/pypi/chishop)
  a django based server, which also allows uploads

simplepypi (http://pypi.python.org/pypi/simplepypi)
  a twisted based solution, which allows uploads

ClueReleaseManager (http://pypi.python.org/pypi/ClueReleaseManager)
  Werkzeug based solution, allows uploads

haufe.eggserver (http://pypi.python.org/pypi/haufe.eggserver)
  GROK/Zope based, allows uploads

scrambled (http://pypi.python.org/pypi/scrambled)
  doesn't require external dependencies, no uploads.

EggBasket (http://pypi.python.org/pypi/EggBasket)
  TurboGears based, allows uploads


Changelog
=========
0.6.1 (2012-08-07)
------------------
- make 'python setup.py register' work
- added init scripts to start pypiserver on ubuntu/opensuse

0.6.0 (2012-06-14)
------------------
- make pypiserver work with pip on windows
- add support for password protected uploads
- make pypiserver work with non-root paths
- make pypiserver 'paste compatible'
- allow to serve multiple package directories using paste

0.5.2 (2012-03-27)
------------------
- provide a way to get the WSGI app
- improved package name and version guessing
- use case insensitive matching when removing archive suffixes
- fix pytz issue #6

0.5.1 (2012-02-23)
------------------
- make 'pypi-server -U' compatible with pip 1.1

0.5.0 (2011-12-05)
------------------
- make setup.py install without calling 2to3 by changing source code
  to be compatible with both python 2 and python 3. We now ship a
  slightly patched version of bottle. The upcoming bottle 0.11
  also contains these changes.
- make the single-file pypi-server-standalone.py work with python 3

0.4.1 (2011-11-23)
------------------
- upgrade bottle to 0.9.7, fixes possible installation issues with
  python 3
- remove dependency on pkg_resources module when running
  'pypi-server -U'

0.4.0 (2011-11-19)
------------------
- add functionality to manage package updates
- updated documentation
- python 3 support has been added

0.3.0 (2011-10-07)
------------------
- pypiserver now scans the given root directory and it's
  subdirectories recursively for packages. Files and directories
  starting with a dot are now being ignored.
- /favicon.ico now returns a "404 Not Found" error
- pypiserver now contains some unit tests to be run with tox

0.2.0 (2011-08-09)
------------------
- better matching of package names (i.e. don't install package if only
  a prefix matches)
- redirect to the real pypi.python.org server if a package is not found.
- add some documentation about configuring easy_install/pip

0.1.3 (2011-08-01)
------------------
- provide single file script pypi-server-standalone.py
- better documentation

0.1.2 (2011-08-01)
------------------
- prefix comparison is now case insensitive
- added usage message
- show minimal information for root url

0.1.1 (2011-07-29)
------------------
- don't require external dependencies

0.1.0 (2011-07-29)
------------------
- initial release


.. _bottle: http://bottlepy.org
.. _PyPI: http://pypi.python.org
.. _pyg: http://pypi.python.org/pypi/pyg
