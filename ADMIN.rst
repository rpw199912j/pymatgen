Introduction
============

This documentation provides a guide for pymatgen administrators. The following
assumes you are using miniconda or Anaconda.

Releases
========

The general procedure to releasing pymatgen comprises the following steps:

1. Wait for all unittests to pass on CircleCI.
2. Update and edit change log.
3. Release PyPI versions + doc.
4. Release conda versions.
5. Release Dash documentation.

Initial setup
-------------

Install some conda tools first::

	conda install --yes conda-build anaconda-client
	conda config --add channels matsci

Pymatgen uses `invoke <http://www.pyinvoke.org/>`_ to automate releases. You will
also need sphinx and doc2dash. Install these using::

	pip install --upgrade invoke sphinx doc2dash

For 2018, we will release both py27 and py37 versions of pymatgen. Create
environments for py27 and py37 using conda::

	conda create --yes -n py37 python=3.7
	conda create --yes -n py27 python=2.7

For each env, install some packages using conda followed by dev install for
pymatgen::

	conda activate py37
	conda install --yes numpy scipy sympy matplotlib cython
	pip install -r requirements.txt
	pip install -r requirements-optional.txt
	pip install invoke sphinx doc2dash
	python setup.py develop
	conda activate py27
	conda install --yes numpy scipy sympy matplotlib cython
	pip install -r requirements.txt
	pip install -r requirements-optional.txt
	pip install invoke sphinx doc2dash
	python setup.py develop

Add your PyPI username and password and GITHUB_RELEASE_TOKEN into your
environment::

	export TWINE_USERNAME=PYPIUSERNAME
	export TWINE_PASSWORD=PYPIPASSWORD
	export GITHUB_RELEASES_TOKEN=TOKEN_YOU_GET_FROM_GITHUB

You may want to add these to your .bash_profile to avoid having to type these
each time.

Machine-specific issues
~~~~~~~~~~~~~~~~~~~~~~~

The above instructions are general, but there are some known issues that are
machine-specific:

* Installing lxml via pip required `STATIC_DEPS=true pip install lxml` on
  macOS 10.13.
* It can be useful to `pip install --upgrade pip twine setuptools` (this may
  be necessary if there are authentication errors when connecting to PyPI).
* You may have to `brew install hdf5 netcdf` or similar to be able to pip
  install the optional requirement `netCDF4`.

Doing the release
-----------------

Ensure appropriate environment variables are set including `DISCOURSE_API_USERNAME`,
`DISCOURSE_API_KEY` and `GITHUB_RELEASES_TOKEN`.

First update the change log. The autogenerated change log is simply a list of
commit messages since the last version.  Make sure to edit the log for brevity
and to attribute significant features to appropriate developers::

    conda activate py37
    invoke update-changelog

Then, do the release with the following sequence of commands (you can put them
in a bash script in your PATH somewhere)::

    conda activate py37
    invoke release --notest --nodoc
    conda deactivate
    invoke update-doc
    conda deactivate
    python setup.py develop

Double check that the releases are properly done on Pypi. If you are releasing
on a Mac, you should see a pymatgen.version.tar.gz and two wheels (Py37 and
P). There will be a py37 wheel for Windows that is generated by Appveyor.

Materials.sh
------------

Fork and clone the `materials.sh <https://github.com/materialsvirtuallab/materials.sh>`_.
This repo contains the conda skeletons to build the conda versions for various
matsci codes on the Anaconda `matsci channel <https://anaconda.org/matsci>`_.

The first time this is run, you may need to `pip install beautifulsoup4`.

If you doing this for the first time, make sure conda-build and anaconda-client
are installed::

	conda install --yes conda-build anaconda-client

Update the pymatgen meta.yaml::

	invoke update-pypi pymatgen

Build the mac versions manually::

	invoke build-conda pymatgen

Commit and push to repo, which will build the Linux and Windows versions.

Check that the `matsci channel <https://anaconda.org/matsci>`_ versions are
properly updated.

Dash docs
---------

Fork and clone the `Dash User Contributions repo <https://github.com/Kapeli/Dash-User-Contributions>`_.

Generate the offline Dash doc using::

	invoke contribute-dash

Create a pull request and submit.
