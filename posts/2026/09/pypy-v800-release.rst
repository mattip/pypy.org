.. title: PyPy v78.0.0 release
.. slug: pypy-v800-release
.. date: 2026-09-18 11:00:00 UTC
.. tags: release
.. category: 
.. link: 
.. description: 
.. type: rst
.. author: mattip

======================================================================
PyPy v8.0.0: release of python 2.7, 3.11,3.12 beta released 2026-09-19
======================================================================

The PyPy team is proud to release version 8.0.0 of PyPy after the previous
release on May 26, 2026. This is a major new version, hence the bump to 8.0.0.
It is our first release of Python 3.12, which may still have some bugs so we
are calling it "beta" quality. 

Why the move to 8.0.0
=====================

glibc2.28
---------

We have updated our linux buildbots (linux64, linux32, aarch64) to use
manylinux_2_28 images based on AlmaLinux 8 and glibc 2.28. These use gcc14
instead of the gcc5 previously used.  So our compiled tarballs will require at
least glibc2.28, which should be universally supported by now (Ubuntu 24.04
uses glibc2.39). In order to prevent confusion, we felt bumping the major
version would be prudent.

cp12-abi3 support
-----------------

PyPy's Python3.12 support comes with a new model for the C layer ``PyObject``.
In order to link the C object to the internal RPython one, we have an extra
field in the object ``ob_pypy_link``, as described in-depth in
`rawrefcount-and-the-gc`_. In previous versions, this field was
visible in a way that makes the ``PyObject`` struct different from the CPython
one. From v8.0.0, we "hide" the PyPy-only extension in a prefix before
the pointer we hand off to C-extension modules. The goal of this work is to
allow PyPy to use cp312-abi3 wheels produced for CPython 3.12 and up, using the
limited ABI. The required pieces have all been put in place:

- PyPy's C headers, including struct definitions like ``PyObject``, are
  compatible with CPython's C headers when defining
  ``Py_LIMITED_API=0x030C0000``
- PyPy no longer mangles exported function names from the limited API.
  In PyPy3.11 and earlier, functions like ``PyTuple_New`` were exported as
  ``PyPyTupleNew``. 

Still missing: the import machinery must be taught that abi3.so shared objects
are valid for PyPy, and the larger ecosystem (pip, uv) must also accept that
cp312-abi3 wheels are valid candidates for installation.

Yes, this is a big step. We are working with Cython and PyO3 to make sure it
all will Just Work™. Hopefully this will make it easier for packages to
support PyPy.
  
.. _`rawrefcount-and-the-gc`: https://doc.pypy.org/discussion/rawrefcount.html


What is new in RPython code generation
=======================================

PyPy is written in RPython, and has code generation to translate RPython into
C as part of the VM build process. We have made some improvements to code
generation in attempts to speed up the base interpreter. While the speedups
have not been that impressive, we have made some steps forward:

- We now use `computed gotos`_ and more aggressively inline code. While this
  produces more compact sources, it does not boost performance as much as we
  wished.

- The source code includes comments mapping the source back to the RPython code
  that generated the block. This is very helpful to see exactly what is going on,
  and may enable further improvements.

Dropping HPy
============

We have dropped the internal `HPy`_ backend for PyPy. The HPy project's
understanding of how to use handles instead of pointers was a good prototype,
but the project did not attract enough supporters to become a new standard. The
code is still in the PyPy codebase, and can be toggled on with a `build option`_.

.. _`build option`: https://doc.pypy.org/config/commandline.html#pypy-python-interpreter-options

A revived tool comparing headers and exported functions
=======================================================

We revived the `clang-based pyhdrdump`_ to compare PyPy's header files to
CPython's header files. See `the README`_ for more information on how it works
and how to use it.

Interpreters
============

The release includes three different interpreters:

- PyPy2.7, supporting the syntax and the features of
  Python 2.7 including the stdlib for CPython 2.7.18+ (the ``+`` is for
  backported security updates)

- PyPy3.11, supporting the syntax and the features of
  Python 3.11, including the stdlib for CPython 3.11.16. Barring security
  issues, this will be the last release to support 3.11.

- PyPy3.12, supporting the syntax and features of Python3.12, including the
  stdlib for CPython 3.12.14.

The interpreters are based on much the same codebase, thus the triple
release.

We recommend updating. You can find links to download the releases here:

    https://pypy.org/download.html

We would like to thank our donors for the continued support of the PyPy
project. If PyPy is not quite good enough for your needs, we are available for
`direct consulting`_ work. If PyPy is helping you out, we would love to hear
about it and encourage submissions to our blog_ via a pull request
to https://github.com/pypy/pypy.org

We would also like to thank our contributors and encourage new people to join
the project. PyPy has many layers and we need help with all of them: bug fixes,
`PyPy`_ and `RPython`_ documentation improvements, or general `help`_ with
making RPython's JIT even better.

If you are a python library maintainer and use C-extensions, please consider
making a CFFI_ version of your library that would be performant
on PyPy. Failing that, PyPy will soon support the cp312-abi3 tag for limited
ABI wheels .  In any case, `cibuildwheel`_ supports building wheels for PyPy.

.. _`PyPy`: https://doc.pypy.org/
.. _`RPython`: https://rpython.readthedocs.org
.. _`help`: https://doc.pypy.org/project-ideas.html
.. _CFFI: https://cffi.readthedocs.io
.. _`cibuildwheel`: https://github.com/joerick/cibuildwheel
.. _blog: https://pypy.org/blog
.. _HPy: https://hpyproject.org/
.. _direct consulting: https://www.pypy.org/pypy-sponsors.html
.. _`computed gotos`: https://eli.thegreenplace.net/2012/07/12/computed-goto-for-efficient-dispatch-tables
.. _`the README`: https://github.com/pypy/pypy/tree/py3.12/pypy/tool/pyhdrdump#pyhdrdump

What is PyPy?
=============

PyPy is a Python interpreter, a drop-in replacement for CPython.
It's fast (`PyPy and CPython`_ performance
comparison) due to its integrated tracing JIT compiler.

We also welcome developers of other `dynamic languages`_ to see what RPython
can do for them.

We provide binary builds for:

* **x86** machines on most common operating systems
  (Linux 32/64 bits, Mac OS 64 bits, Windows 64 bits)

* 64-bit **ARM** machines running Linux (``aarch64``) and macos (``macos_arm64``).

PyPy supports Windows 32-bit, Linux PPC64 big- and little-endian, Linux ARM
32 bit, RISC-V RV64IMAFD Linux, and s390x Linux but does not release binaries.
Please reach out to us if you wish to sponsor binary releases for those
platforms. Downstream packagers provide binary builds for debian, Fedora,
conda, OpenBSD, FreeBSD, Gentoo, and more.

.. _`PyPy and CPython`: https://speed.pypy.org
.. _`dynamic languages`: https://rpython.readthedocs.io/en/latest/examples.html

What else is new?
=================

For more information about the 8.0.0 release, see the `full changelog`_.

Please update, and continue to help us make pypy better.

Cheers,
The PyPy Team

.. _`full changelog`: https://doc.pypy.org/en/latest/release-v8.0.0.html#changelog 
