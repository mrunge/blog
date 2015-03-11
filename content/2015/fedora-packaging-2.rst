Fedora packaging workshop
#########################
:date: 2015-02-07 21:32
:author: mrunge
:category: Fedora
:tags: Fedora, packaging
:slug: fedora-packaging-2

This part assumes, you already have the tools installed like described
in `earlier part`_.

Using a tool to create a spec
-----------------------------

As first example, we'll use a tool to create a spec file, download
the sources.

.. code-block:: bash

  rpmdev-setuptree
  cd ~/rpmbuild
  pyp2rpm -n bleach > SPECS/python-bleach.spec

What this actually does is to create a file structure for your builds and
change into that directory.
Finally ``pyp2rpm`` goes to `pypi`_, pulls down the required information,
downloads the sources, copies them to ``~rpmbuild/SOURCES`` and writes
as much information as possible to the spec file.

.. code-block:: spec

  # Created by pyp2rpm-1.1.2
  %global pypi_name bleach
  
  Name:           python-%{pypi_name}
  Version:        1.4.1
  Release:        1%{?dist}
  Summary:        An easy whitelist-based HTML-sanitizing tool
  
  License:        ASL %(TODO: version)s
  URL:            http://github.com/jsocol/bleach
  Source0:        https://pypi.python.org/packages/source/b/%{pypi_name}/%{pypi_name}-%{version}.tar.gz
  BuildArch:      noarch
 
  BuildRequires:  python2-devel
  BuildRequires:  python-nose >= 1.3
 
  Requires:       python-six
  Requires:       python-html5lib >= 0.999

  %description
  Bleach is an HTML sanitizing library
  that escapes or strips markup and
  attributes based on a white list. 

  %prep
  %setup -q -n %{pypi_name}-%{version}
  # Remove bundled egg-info
  rm -rf %{pypi_name}.egg-info

  %build
  %{__python2} setup.py build

  %install
  %{__python2} setup.py install --skip-build --root %{buildroot}

  %check
  %{__python2} setup.py test

  %files
  %doc README.rst LICENSE
  %{python2_sitelib}/%{pypi_name}
  %{python2_sitelib}/%{pypi_name}-%{version}-py?.?.egg-info

  %changelog
  * Sat Feb 07 2015  - 1.4.1-1
  - Initial package.
  

Magic, isn't it? Unfortunately, that's not completely everything:

- Look at the license field. Actually, the license is `ASL 2.0`_.
  To make it build, you'll also need ``python-setuptools``.
- The license should be marked as %license and not as %doc
- run time requirements are required for testing as well. We should add them.
  The builders are not connected to the internet, thus something like
  ``pip install`` does not work during package build or testing.
- The changelog entry should contain a name and an email-address.

Once that is fixed, let's issue a local build:

.. code-block:: shell

  rpmbuild -ba SPECS/python-bleach.spec

which will building a binary package and a Source RPM package.

Finally, one should try a mock build, which will build the same package
in a clean environment. This will discover any forgotten dependencies.

.. code-block:: shell

  mock --rebuild ./SRPMS/python-bleach-1.4.1-1.fc21.src.rpm

Now that is finished, we're nearly done. It's time to check for anything
other obviously forgotten:

.. code-block:: shell

  [rpmbuilder@turing rpmbuild]$ rpmlint ./SPECS/python-bleach.spec ./RPMS/noarch/python-bleach-1.4.1-1.fc21.noarch.rpm ./SRPMS/python-bleach-1.4.1-1.fc21.src.rpm 
  python-bleach.noarch: W: spelling-error Summary(en_US) whitelist -> white list, white-list, whistle
  python-bleach.src: W: spelling-error Summary(en_US) whitelist -> white list, white-list, whistle
  2 packages and 1 specfiles checked; 0 errors, 2 warnings.


``rpmlint`` is a quite powerful linter to check for common issues.

The finished spec for python-bleach can be found here: `http://www.matthias-runge.de/fedora/python-bleach.spec`_.

.. _`earlier part`: http://www.matthias-runge.de/2014/10/08/fedora-packaging/
.. _`pypi`: https://pypi.python.org/pypi
.. _`ASL 2.0`: https://github.com/jsocol/bleach/blob/master/LICENSE
.. _`http://www.matthias-runge.de/fedora/python-bleach.spec`: http://www.matthias-runge.de/fedora/python-bleach.spec
