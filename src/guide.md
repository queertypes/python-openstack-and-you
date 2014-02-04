% Python, Openstack, and You
% Alejandro Cabrera (@cppcabrera, irc: alcabrera)
% February 4, 2014

\pagebreak

Alpha preview: Notes forthcoming on many sections below.

```
>>> import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

![](cc-by-sa.png "CC-BY-SA 4.0")

Python, Openstack, and You by Alejandro Cabrera is licensed under a
Creative Commons Attribution-ShareAlike 4.0 International License.

\pagebreak

## Preface {#preface}

Hello, reader! This guides aims to help increase your understanding of
the Python development ecosystem. The ultimate goal is to help you
become sufficiently proficient at Python that: 1) you can build your
own packages, 2) become an effective contributor to open source
projects.

Python is a dynamically-typed language with very expressive
features. Users familiar with C-like languages will feel at home in
Python. The community behind the language is amazing, vocal, and
supportive.

This guide also aims to introduce you to the Openstack developer
ecosystem. Therein, I hope to have you ready to contribute to the
project soon after completing this guide.

Python and Openstack: the two go hand-in-hand. The skills that you
acquire while working in Python will benefit you in the world of
Openstack. The time you spend with Openstack will help you understand
how Python works in large projects, and how to collaborate with
others.

Enjoy the guide! Reach out to me at
[@cppcabrera](http://twitter.com/cppcabrera) or on freenode:alcabrera
if you'd like to improve this guide.

## Project Structure {#structure}

### High-Level View

```bash
marconi
|-- AUTHORS
|-- AUTHORS.rst
|-- ChangeLog
|-- etc  # configuration file samples, usually
|   |-- logging.conf-sample
|   `-- marconi.conf-sample
|-- HACKING.rst  # Contributors guide
|-- MANIFEST.in  # Rules for auto-packaging; more later
|-- marconi  # main project source
|   |-- cmd
|   |-- common
|   |-- __init__.py  # for python 2.x, identifies dir as a module
|   |-- __init__.pyc
|   |-- openstack
|   |-- __pycache__
|   |-- queues
|   |-- tests
|   |-- version.py
|   `-- version.pyc
|-- openstack-common.conf
|-- README.rst  # front page documentation
|-- requirements.txt  # third party dependencies
|-- setup.cfg  # new-style package metadata
|-- setup.py   # package generating script; usually simple
|-- test-requirements.txt  # third party deps for testing
|-- tests  # all test code
|   |-- etc
|   |-- functional
|   |-- __init__.py
|   `-- unit
`-- tox.ini  # tox config; more on this later
```

\pagebreak

### Source Organization

### Test Organization

### Documentation Organization

### Configuration Files

### Binaries/Scripts

### Other Files

## Packaging {#packaging}

### Tools {#tools}

#### Pip: One-stop Shop for Python Module Installation {#pip}

#### A Tale of Two Python Packaging Branches {#setuptools-distutils}

### Package Formats {#formats}

#### Eggs {#eggs}

#### Wheels {#wheels}

### Make Your Own Python Package {#twine}

#### Making a setup.cfg

#### More From Your Manifest

#### Installing Binaries/Scripts

#### Getting on PyPI {#pypi}

##### Register with PyPI {#pypi-register}

##### Generate the Things {#pypi-dist}

##### Upload to PyPI {#pypi-upload}

## Development {#developing}

### Use These

* ipython
* virtualenv
* tox
* pip
* pbr
* flake8
* hacking
* pylint
* nosetests

### Python Language Features: Not Covered Here

Python is an extremely expressive language. In the interest of
brevity, know that the things listed below are powerful, flexible, and
mostly composable, but will not be covered here.

* basics: operators, syntax, etc.
* (list, set, dict) comprehensions
* generators
* iteration
* decorators
* context managers
* function argument passing techniques (a, b, *args, **kwargs)
* deconstructing assignments (```a, b, c = (1, 2, 3)```)
    * ...and the py3k variant: ```a, *b, c = range(10)```
* python 3 annotations
* exception handling
* docstrings

## Supporting Python 2.6, 2.7, 3.3 and Beyond {#python2-to-3}

### From \_\_future\_\_ Import {#future}

#### Print is a Function {#print-function}

#### Unicode Literals {#unicode-literals}

#### Division is Floating Point by Default {#float-div}

#### New Exception-Handling Syntax {#exceptions}

### Python 3 Backports {#py3-backports}

#### Enums

#### Concurrent.Futures

### Only in Python 3 {#py3-specific}

#### yield from

####

### Checking Whether Your Code Runs on Python 3 {#my-py3}

### Checking Whether a Dependency Runs on Python 3 {#dep-py3}

* Does the dependency run on python 3.x?
    * step 1: can you 'pip install' it?
    * step 2: does it advertise python 3 on PyPI?
    * step 3: does its unit tests pass?
    * step 4: when plugged into your code, do your unit tests pass?

\pagebreak

## Testing {#python-testing}

## Libraries {#libs}

### Working With the Batteries {#stdlib}

* json
* multiprocessing
* wsgiref

### Useful Third Party Libraries {#libraries}

* compat
    * six
    * pbr
* web development
    * falcon
    * requests
    * gunicorn
* testing
    * mock
    * httpretty
    * ddt
* database engines
    * sqlalchemy
    * pymongo
    * redis
* others
    * lxml
    * stevedore

### Libraries You Should Know (for Openstack) {#libraries-openstack}

* oslo.config
* openstack.common:
    * logging
* pecan

## Interpreters {#interpreters}

### CPython {#cpython}

### pypy {#pypy}

### Others {#interpret-others}

* jython
* ironpython

## Welcome to Openstack {#openstack}

### Contributing Overview {#openstack-overview}

#### Sign Up with Launchpad {#openstack-launchpad}

#### Sign the Contributor's License Agreement {#openstack-cla}

#### Code Review {#openstack-code-review}

#### Core and Regular Reviewers {#openstack-reviewers}

#### On Style {#openstack-hacking}

## Useful Resources

### General Purpose

1. [Hitchiker's Guide to Python](http://docs.python-guide.org/en/latest/)
1. [Idiomatic Python](http://www.jeffknupp.com/blog/2012/10/04/writing-idiomatic-python/)
1. [PyPI](https://pypi.python.org/pypi)
1. [Python Videos](http://pyvideo.org/)

### On Packaging Formats

1. [Python on Wheels](http://lucumr.pocoo.org/2014/1/27/python-on-wheels/)

### On Python 3

1. [Does My Dependency Support Python 3?](https://python3wos.appspot.com/)
1. [Porting to Python 3 Redux](http://lucumr.pocoo.org/2013/5/21/porting-to-python-3-redux/)
1. [Python 3: Common Stumbling Blocks](http://docs.python.org/3.4/whatsnew/3.0.html#common-stumbling-blocks)
1. [Python 3: Syntax Changes](http://docs.python.org/3.4/whatsnew/3.0.html#overview-of-syntax-changes)
1. [What's New in Python 3.4?](http://docs.python.org/3.4/whatsnew/3.4.html#summary-release-highlights)
1. [Python 3.3: Trust Me, It's Better Than 2.7](http://pyvideo.org/video/1730/python-33-trust-me-its-better-than-27)
1. [asyncio](http://www.youtube.com/watch?v=aurOB4qYuFM&feature=youtu.be)

### On Openstack

1. [Getting Started](https://wiki.openstack.org/wiki/Getting_Started)
1. [Gerrit Workflow](https://wiki.openstack.org/wiki/Gerrit_Workflow)
1. [Icehouse Release Schedule](https://wiki.openstack.org/wiki/Icehouse_Release_Schedule)
1. [GIT Commit Messages](https://wiki.openstack.org/wiki/GitCommitMessages)
1. [Marconi Wiki](https://wiki.openstack.org/wiki/Marconi)
