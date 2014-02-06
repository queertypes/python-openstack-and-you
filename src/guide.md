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

***

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

WSGI is the Web Server Gateway Interface. It makes it possible to
write a web application in Python that can be run on a WSGI server of
your choice. A WSGI application receives HTTP requests and yields HTTP
responses.

This section will cover:

* The basics of WSGI
* How to deploy a ```Hello, World``` WSGI application
* How to use a WSGI framework to provide routing, etc.

### WSGI Servers

A WSGI server implements the WSGI
[specification](http://www.python.org/dev/peps/pep-3333/) and can run
your Python code as a WSGI application.

Some examples of WSGI servers include:

* [gunicorn](http://gunicorn.org/)
* [uwsgi](http://uwsgi-docs.readthedocs.org/en/latest/)
* [chaussette](https://chaussette.readthedocs.org/en/1.0/)
* stdlib: [wsgiref](http://docs.python.org/2/library/wsgiref.html)

I'll be using gunicorn for deployment examples in this section. Let's
install it now:

```bash
$ pip install gunicorn
```

While we're at it, let's bring in the excellent
[HTTPie](https://github.com/jkbr/httpie) tool in, as well, to help us
tinker with our apps.

```bash
$ pip install httpie
```

#### Hello, WSGI

```python
# hello.py
def app(env, start_response):
    start_response('200 OK', [])
    return [b'Hello, world!']
```

That's it. That's the minimum WSGI application. To run it:

```bash
$ gunicorn hello:app
2014-02-06 01:18:18 [12476] [INFO] Starting gunicorn 18.0
2014-02-06 01:18:18 [12476] [INFO] Listening at: http://127.0.0.1:8000 (12476)
2014-02-06 01:18:18 [12476] [INFO] Using worker: sync
2014-02-06 01:18:18 [12481] [INFO] Booting worker with pid: 12481
```

Let's interact!

```bash
$ http get localhost:8000
```

```http
HTTP/1.1 200 OK
Connection: close
Date: Thu, 06 Feb 2014 06:19:36 GMT
Server: gunicorn/18.0
Transfer-Encoding: chunked

Hello, world!
```

### More on WSGI Applications

Let's take another look at that ```Hello, world```.

```python
# hello.py
def app(env, start_response):
    start_response('200 OK', [])
    return [b'Hello, world!']
```

The ```env``` is a dictionary that contains HTTP headers and
information about the WSGI environment. ```start_response``` is a
callback - its full signature looks like:

```python
start_response(
    status: str,
    response_headers: [(str, str)],
    exc_info=None: (type, value, callback)
)
```

* ```status``` - HTTP status, like ```200 OK```, ```404 Not Found```
* ```response_headers``` - key:value pairs of headers, like
  ```('content-type', 'application/json')```
* ```exc_info``` - exception information

Let's print out the ```env``` and see what's inside:

```python
# hello.py
import pprint


def app(env, start_response):
    pprint.pprint(env)
    start_response('200 OK', [])
    return [b'Hello, world!']
```

After reloading ```gunicorn``` and issuing an ```http get
localhost:8000```, we see on the gunicorn console log:

```python
{'HTTP_ACCEPT': '*/*',
 'HTTP_ACCEPT_ENCODING': 'gzip, deflate, compress',
 'HTTP_HOST': 'localhost:8000',
 'HTTP_USER_AGENT': 'HTTPie/0.6.0',
 'PATH_INFO': '/',
 'QUERY_STRING': '',
 'RAW_URI': '/',
 'REMOTE_ADDR': '127.0.0.1',
 'REMOTE_PORT': '33333',
 'REQUEST_METHOD': 'GET',
 'SCRIPT_NAME': '',
 'SERVER_NAME': 'localhost',
 'SERVER_PORT': '8000',
 'SERVER_PROTOCOL': 'HTTP/1.1',
 'SERVER_SOFTWARE': 'gunicorn/18.0',
 'gunicorn.socket': <socket._socketobject object at 0x1a9a600>,
 'wsgi.errors': <open file '<stderr>', mode 'w' at 0x7fe7dc7d01e0>,
 'wsgi.file_wrapper': <class gunicorn.http.wsgi.FileWrapper at 0x1a8a7a0>,
 'wsgi.input': <gunicorn.http.body.Body object at 0x1b0c050>,
 'wsgi.multiprocess': False,
 'wsgi.multithread': False,
 'wsgi.run_once': False,
 'wsgi.url_scheme': 'http',
 'wsgi.version': (1, 0)}
```

I won't cover most of the fields in there. The key takeaways here are:

* WSGI apps are just Python programs
* Checking out state with prints helps with understanding what's going
  on
* HTTP headers take the form ```HTTP_<NAME>```
    * For example, ```X-Auth-Token``` becomes ```HTTP_X_AUTH_TOKEN```

### WSGI Frameworks

Frameworks give you the ability to control your WSGI application more
easily. Most of them wrap the WSGI ```env``` in a ```Request``` object
and allow you to respond by modifying a ```Response```
object. Furthermore, most of them allow you to define routes.

Some WSGI frameworks include:

* [falcon](http://falconframework.org/)
* [pecan](http://pecanpy.org/)
* [django](http://www.djangoproject.com/)
* [flask](http://flask.pocoo.org/)

They vary in the spectrum of tools that they provide. Some, like
Django, are more full-stack oriented, coming with many built-ins
including an ORM. Others, like Falcon and Flask, are minimalist, and
provide at least ```Request``` and ```Response``` objects coupled with
routing.

We'll be using Falcon in the following example. Let's get that installed:

```bash
$ pip install falcon
```

#### Hello, Falcon

```python
# hello_falcon.py
from __future__ import print_function
import falcon


class HelloResource(object):
    def __init__(self, arg):
        print('Initialzing with', arg)

def on_get(self, request, response):
        print([n for n in dir(request) if not n.startswith('_')])
        print([n for n in dir(response) if not n.startswith('_')])
        response.status = falcon.HTTP_200
        response.content_type = 'application/text'
        response.body = 'Hello, world!'


app = falcon.API()
app.add_route('/', HelloResource(1))
```

This application uses Falcon and adds a few features over the original
```Hello, world```. Namely:

* prints out the members of the request object
* prints out the members of the response object
* sets the HTTP status and content-type
* defines only the GET method
* responds with ```Hello``` only on the ```/``` route
* demonstrates trivial responder initialization

Let's see it in action. Load up ```hello_falcon``` in ```gunicorn```
and get ready to ```HTTPie```!

```
$ http get localhost:8000/
```

```http
HTTP/1.1 200 OK
Connection: close
Date: Thu, 06 Feb 2014 06:50:32 GMT
Server: gunicorn/18.0
content-length: 13
content-type: application/text

Hello, world!
```

```
$ http get localhost:8000/notfound
```

```http
HTTP/1.1 404 Not Found
Connection: close
Date: Thu, 06 Feb 2014 06:50:55 GMT
Server: gunicorn/18.0
content-length: 0
```

```
$ http post localhost:8000/
```

```http
HTTP/1.1 405 Method Not Allowed
Connection: close
Date: Thu, 06 Feb 2014 06:51:42 GMT
Server: gunicorn/18.0
allow: GET, OPTIONS
content-length: 0
```

On the gunicorn console, we'll see the following output:

```python
['accept', 'app', 'auth', 'client_accepts', 'client_accepts_json',
 'client_accepts_xml', 'client_prefers', 'content_length',
 'content_type', 'date', 'env', 'expect', 'get_header',
 'get_param', 'get_param_as_bool', 'get_param_as_int',
 'get_param_as_list', 'headers', 'if_match',
 'if_modified_since', 'if_none_match', 'if_range',
 'if_unmodified_since', 'log_error', 'method', 'path',
 'protocol', 'query_string', 'range', 'relative_uri',
 'stream', 'uri', 'url', 'user_agent']
['body', 'body_encoded', 'cache_control', 'content_location',
 'content_range', 'content_type', 'data', 'etag',
 'last_modified', 'location', 'retry_after',
 'set_header', 'set_headers', 'status', 'stream',
 'stream_len', 'vary']
```

The first list are the methods for the ```Request``` object. The
second list shows the members of the ```Response``` object. I won't go
into detail in this guide. The takeaway is:

* Make the most of exploring your framework of choice
    * Use prints
    * Use the python REPL, if you can
    * Read the Docs

Using a WSGI framework makes it much easier to grow your web
application. Best practice here: use a WSGI framework.

#### WSGI Middleware

\  

A lesser known and slightly more advanced feature of the WSGI protocol
is that you can wrap a WSGI application in another WSGI
application. It's
[functional composition](http://mathieularose.com/function-composition-in-python/). This
allows you to add features on top of an existing WSGI application in a
modular fashion.

As an example, let's add logging for ```x-project-id```s:

```python
from __future__ import print_function
import uuid

import falcon


# This is the WSGI middleware
def wraps(app):
    print('Initializing middleware...')
    def logger(env, start_response):
        print(env.get('HTTP_X_PROJECT_ID'))
        return app(env, start_response)
    return logger


# as before
class HelloResource(object):
    def __init__(self, arg):
        print('Initialzing with', arg)

def on_get(self, request, response):
        response.status = falcon.HTTP_200
        response.content_type = 'application/text'
        response.body = 'Hello, world!'


app = falcon.API()
app.add_route('/', HelloResource(1))
app = wraps(app)
```

This gives our WSGI app the ability to "log" the x-project-id sent
with a request. Let's give it a try:

```
$ http get localhost:8000/
...
$ http get localhost:8000/ x-project-id:awesome_user
```

Over on the gunicorn console:

```
None
awesome_user
```

The key takeaway here:

* Think of middleware as function composition
    * Each layer can add a new feature to your application w/o tight coupling

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
