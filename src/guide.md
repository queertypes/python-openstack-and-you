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

### You

This guide assumes that you've at least dabbled with Python before. If
you can load up an interpreter and define some functions, you're
probably good to go.

I also assume you've worked with a command line before. If you can
navigate around with ```cd```, ```ls```, and more, you're probably
good in this area.

### Contributing

Enjoy the guide! Reach out to me at
@[cppcabrera](http://twitter.com/cppcabrera) or on freenode:alcabrera
if you'd like to give feedback, or fork away and send
[pull requests](https://github.com/cabrera/python-openstack-and-you)!

\newpage

## Project Structure {#structure}

After you've worked on Python for some time, certain patterns about
project management start to become intuitive. This section hopes to
distill that intuition.

Python projects usually consist of several common files. Below is a
```tree```-listing of the
[Openstack Marconi](https://github.com/openstack/marconi) project.

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

It's fairly common to have the primary source directory named after
the project itself. This comes with the bonus of avoiding module
collisions for free if you choose to upload your package to
[PyPI](https://pypi.python.org/pypi).

Inside each source folder, be sure to include an
```__init__.py```. Even if the file is empty, it controls whether
python 2.7 - 3.2 recognize the given directory as a module. Starting
with Python 3.3, this is no longer necessary. This ```__init__.py```
can also be used to trigger import-time side-effects.

For example, in marconi.queues.storage:

```python
# marconi/queues/storage/__init__.py
"""Marconi Storage Drivers"""

from marconi.queues.storage import base
from marconi.queues.storage import errors  # NOQA

# Hoist classes into package namespace
ControlDriverBase = base.ControlDriverBase
DataDriverBase = base.DataDriverBase
CatalogueBase = base.CatalogueBase
Claim = base.Claim
Message = base.Message
Queue = base.Queue
ShardsBase = base.ShardsBase
```

This makes it so if someone imports ```marconi.queues.storage``` into
their module, the chosen names from the sub-modules become available.

This is mostly a convenience to users. Best practices to keep in mind:

* Avoid modifying global state using import-time side-effects
* Consider hoisting certain names as a convenience to users

\pagebreak

### Test Organization

It's great to start organizing your tests early on. A frequently used
approach to this is:

* ```unit```: tests that avoid/mock network/disk I/O and randomness
    * The primary goal of unit testing is to be your "compiler" - can my code run?
    * Can also serve to verify properties of [pure](http://en.wikipedia.org/wiki/Pure_function) functions
* ```functional```: tests that perform network/disk I/O
    * Primary goal: verify operating correctness of your system

We'll dive a little deeper into the python testing ecosystem
[later](#python-testing).

### Documentation Organization

There's a great deal of information on how to write great
documentation. To be honest, as of this moment, I have close to zero
experience generating documentation for Python projects.

I refer you to the
[python-requests](https://github.com/kennethreitz/requests)
[docs](http://docs.python-requests.org/en/latest/) for an example of
well-written documentation, and to the python-requests
[src](https://github.com/kennethreitz/requests/tree/master/docs) for
how to structure your docs.

### Configuration Files

If you're developing a Python service, it's likely that you'll need to
expose some configuration files. My experiences lie mostly within the
realm of
[oslo.config](http://docs.openstack.org/developer/oslo.config/), so
the advice I give here comes from that. Given that, the advice given
below can be generalized to different, effective configuration
strategies.

\pagebreak

[oslo.config](http://docs.openstack.org/developer/oslo.config/) by
default searches for configuration files in four directories:

```
~/.${project}/
~/
/etc/${project}/
/etc/
```

The ```project``` is determined by how the configuration is
instantiated for your project. For example, in Openstack Marconi, this
is accomplished as follows:

```python
from oslo.config import cfg

from marconi.queues import bootstrap

conf = cfg.CONF
conf(project='marconi', prog='marconi-queues', args=[])

app = bootstrap.Bootstrap(conf).transport.ap
```

Because of ```project='marconi'```, oslo.config will search for
configuration information in:

```
~/.marconi/
~/
/etc/marconi/
/etc/
```

\pagebreak

Configuration options are declared by placing some global declarations
in a python file. For example, the base storage options for Marconi
have the following configurations options defined:

```python
# marconi.queues.storage.base
_LIMITS_OPTIONS = [
    cfg.IntOpt('default_queue_paging', default=10,
               help='Default queue pagination size'),
    cfg.IntOpt('default_message_paging', default=10,
               help='Default message pagination size')
]

_LIMITS_GROUP = 'limits:storage'
```

In order to make them available for use throughout the project, they
must be registered. This is done as follows:

```python
# marconi.queues.storage.base
class DataDriverBase(DriverBase):

    def __init__(self, conf, cache):
        super(DataDriverBase, self).__init__(conf, cache)

        self.conf.register_opts(_LIMITS_OPTIONS, group=_LIMITS_GROUP)
        self.limits_conf = self.conf[_LIMITS_GROUP]
```

The act of ```DataDriverBase``` calling ```register_opts``` achieves
configuration registration. The ```group``` controls the
section. It'll correspond to ```[limits:storage]``` in the final
configuration file.

\pagebreak

Some best practices to keep in mind:

* Declare configuration options in the file that it relates to the most
* Assign reasonable types to your config options
    * An ```int``` probably doesn't make sense for an IP address, for
      example
* Register your configuration options during the initialization of a
  resource
* Avoid global configuration objects as much as possible
    * Passing it around as a function argument allows for more
      testable code

### Binaries/Scripts

Some Python projects install with binaries or scripts. For example,
the [flake8](http://flake8.readthedocs.org/en/latest/) static analyzer
is mostly a script.

It's wise to bundle most of the logic as modules, then import what's
needed and provide that as the script, especially for larger
projects. The logic would be kept in the ```{project}``` directory and
the script would be kept in ```bin/{script}```.

In order to specify that a script be installed along with the library
code for your project, specify it as follows in the ```setup.cfg```:

```ini
[entry_points]
console_scripts =
    marconi-server = marconi.cmd.server:run
```

The name on the left-hand side will be the name given to the
executable when it is installed. The right-hand side corresponds to
the module search path followed by the function to run in that module.

### Other Files

There's a few more files to keep in mind when working with Python
projects. Not all projects will have all of the files listed below,
but it's good to be aware of them:

* setup.py: used by package manager - installation metadata/instructions
* setup.cfg: new-style installation metadata/instructions
* tox.ini: for use with [tox](#tox) - easy, multi-env testing
* MANIFIEST.in: gives instructions to include files at install time
* requirements.txt: list of dependencies for using your package
* test-requirements.txt: list of dependencies for testing

## Packaging {#packaging}

The Python packaging ecosystem is wide and diverse. With no less than
three formats in use at the moment (egg, wheel, pex), it's easy to get
confused.

This section aims to provide you with the knowledge you need to create
and upload your own python package to the Python Package Index
(PyPI). Note that this subject has already been covered in detail by
the excellent
[Hitchiker's Guide to Python](http://docs.python-guide.org/en/latest/shipping/packaging/).

What this guide adds is an introduction to
[Python Build Reasonableness](https://github.com/openstack-dev/pbr)
(pbr), which is in active use within the openstack ecosystem. It aims
to unify the previous existing differences between setuptools and
distribute.

### Tools {#tools}

A few tools make interacting with the Python packaging ecosystem quite
pleasant. In particular, take note of:

* [pbr](#pbr)
* [pip](#pip)
* [twine](#twine)

Each is introduced in detail below. The short version:

* pip allows you to [un]install packages
* pbr unifies the package metadata format
* twine makes uploading to PyPI safer

#### Pip: One-stop Shop for Python Module Installation {#pip}

[pip](http://www.pip-installer.org/en/latest/) is to
[python](http://python.org/) as [gem](http://rubygems.org/) is to
[ruby](https://www.ruby-lang.org/en/), [npm](https://npmjs.org/) is to
[node](http://nodejs.org/)/javascript,
[cabal](http://www.haskell.org/cabal/) is to
[haskell](http://www.haskell.org/haskellwiki/Haskell), and so on.

With a single command, you can...

* install a package: ```pip install redis```
* uninstall a package: ```pip uninstall redis```
* show what packages are installed: ```pip freeze```
* upgrade a package: ```pip install --upgrade redis```
* upgrade pip: ```pip install --upgrade pip```

pip accomplishes all of this by searching
[PyPI](https://pypi.python.org/) for a package with the name you gave
it. Having found the package by name, it will install it. This looks
like:

```bash
$ pip install pep8
Downloading/unpacking pep8
  Downloading pep8-1.4.6.tar.gz (65kB): 65kB downloaded
  Running setup.py egg_info for package pep8

    no previously-included directories found matching 'docs/_build'
Installing collected packages: pep8
  Running setup.py install for pep8

    no previously-included directories found matching 'docs/_build'
    Installing pep8 script to /home/alejandro/.venv/27/bin
Successfully installed pep8
Cleaning up...
```

pip depends on the presence of a setup.py in order to works its
magic. setup.py is the package metadata and includes information like:

* who is the author of this package?
* what license does it use?
* what dependencies does the package have?

Writing a setup.py always defers to an underlying packaging
library. There are a few in circulation, including ```distutils```
which comes with the standard library. This
[stack overflow answer](http://stackoverflow.com/a/14753678)
summarizes the differences between the package libraries. This guide
advocates side-stepping the issue by making use of [pbr](#pbr).

### Make Your Own Python Package {#twine}

#### Making a setup.cfg With pbr {#pbr}

#### setup.py: Another Way to Do It

#### More From Your Manifest

#### Installing Binaries/Scripts

#### Getting on PyPI {#pypi}

##### Register with PyPI {#pypi-register}

##### Generate the Things {#pypi-dist}

##### Upload to PyPI {#pypi-upload}

### Package Formats {#formats}

#### Eggs {#eggs}

#### Wheels {#wheels}

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

Python 2 and Python 3 are rather different beasts. For example, in
Python 2, ```print``` is a statement, whereas in Python 3, it is a
function.

This section will guide you through the process of getting your
project to be compatible with Python 3, while maintaining Python
2.6/2.7 support.

### From \_\_future\_\_ Import {#future}

```__future__``` imports are a convenient mechanism for exposing
back-ported features to older Python versions. Some that are
particularly interesting for maintaining 2/3 compatibility are:

* [print function](#print-function)
* [unicode literals](#unicode-literals)
* [true division](#float-div)
* [absolute imports](#absolute-imports)

A ```__future__``` import must come before any other import.

#### Print is a Function {#print-function}


```python
# Python 2
print >> sys.stderr, "My", "data"

# Python 3
print('My', 'data', file=sys.stderr)

# Compatible approach:
from __future__ import print_function
print('My', 'data', file=sys.stderr)
```

#### Unicode Literals {#unicode-literals}  
\  

String-like data handling is one of the most common causes of
incompatibilities between Python 2 and 3.

In Python 2, there are at least three types of strings: ```bytes```,
```str```, and ```unicode```. In Python 3, there are only ```bytes```
and ```str```. The ```str``` exposed by Python 3 is mostly equivalent
to the Python 2 ```unicode``` type. Python 2's ```str```-type is the
odd one out going between versions. In short:

+----------+----------------+
| Python 2 |    Python 3    |
+==========+================+
| str      | (almost) bytes |
+----------+----------------+
| unicode  | str            |
+----------+----------------+
| bytes    | bytes          |
+----------+----------------+

Table: Strings in Python 2 vs. Python 3

When using the ```unicode_literals``` future import, it makes it so
that string literals occurring in Python 2 source is treated as
unicode string.

For example, in Python 2:

```python
>>> type('aa'), type(u'aa'), type(b'aa')
(<type 'str'>, <type 'unicode'>, <type 'str'>)
>>> from __future__ import unicode_literals
>>> type('aa'), type(u'aa'), type(b'aa')
(<type 'unicode'>, <type 'unicode'>, <type 'str'>)
```

In Python 3:

```python
>>> type('aa'), type(u'aa'), type(b'aa')
(<class 'str'>, <class 'str'>, <class 'bytes'>)
>>> from __future__ import unicode_literals
>>> type('aa'), type(u'aa'), type(b'aa')
(<type 'str'>, <type 'str'>, <type 'bytes'>)
```

To mostly unify the way string literals are handled between Python 2
and 3, use the ```unicode_literals``` import.

#### Division is Floating Point by Default {#float-div}

```python
# Python 2
>>> 2 / 3, 2 // 3, 2.0 / 3, 2.0 // 3
(0, 0, 0.6666666666666666, 0.0)
# Python 3
>>> 2 / 3, 2 // 3, 2.0 / 3, 2.0 // 3
(0.6666666666666666, 0, 0.6666666666666666, 0.0)

# Compatible approach
from __future__ import division
>>> 2 / 3, 2 // 3, 2.0 / 3, 2.0 // 3
(0.6666666666666666, 0, 0.6666666666666666, 0.0)
```

#### New Exception-Handling Syntax {#exceptions}

```python
# Python < 2.6
try:
    thing()
except (Exception, RuntimeError), e:
    print(e)
# Python >= 2.6
try:
    thing()
except (Exception, RuntimeError) as e:
    print(e)
```

Since we're trying to support Python >= 2.6, it makes sense to take
advantage of the ```as``` syntax. In fact, the old syntax causes a
syntax error in Python 3!

#### Absolute Imports



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

## Web Development

### WSGI

### WSGI Servers

#### Hello, WSGI

```python
def app(env, start_response):
    start_response('200 OK', [])
    return [b'Hello, world!']
```

### WSGI Frameworks

Links to various frameworks:

*

#### Hello, Falcon

### WSGI Tools

## Welcome to Openstack {#openstack}

### Contributing Overview {#openstack-overview}

#### Sign Up with Launchpad {#openstack-launchpad}

#### Sign the Contributor's License Agreement {#openstack-cla}

#### Code Review {#openstack-code-review}

#### Core and Regular Reviewers {#openstack-reviewers}

#### On Style {#openstack-hacking}

\pagebreak

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
