# Maintenance Fork of [Asottile's Setuptools Golang Library](https://github.com/asottile-archive/setuptools-golang)

All credit for this library goes to the original author [Anthony Sottile](https://github.com/asottile). This fork is for CVE and minor bug fixes, but it has the following major limitation.

# WARNING 

It turns out multiple go shared objects in a single process is not supported

it likely broke in [go 1.21] and there is no intention to fix it :(

[go 1.21]: https://github.com/golang/go/issues/65050#issue-2074509727

___

[![build status](https://github.com/asottile/setuptools-golang/actions/workflows/main.yml/badge.svg)](https://github.com/asottile/setuptools-golang/actions/workflows/main.yml)
[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/asottile/setuptools-golang/main.svg)](https://results.pre-commit.ci/latest/github/asottile/setuptools-golang/main)

setuptools-golang
=================

A setuptools extension for building cpython extensions written in golang.

## Requirements

This requires golang >= 1.5.

This requires python >= 3.7.  It is currently tested against python3 and pypy3.

## Platform Support

- linux
- macOS
- win32

## Usage

Add `setuptools-golang` to the `setup_requires` in your setup.py and
`build_golang={'root': ...}`.  `root` refers to the root go import path of
your project.

By default, `setuptools-golang` will strip all binaries. This can be disabled
by adding `'strip': False` to `build_golang`. This will increase the size of
the extension, but the binaries contain debugging information and symbols.


An extension must be a single file in the `main` go package (though the entire
`main` package will be built into the extension).  That package may import
other code.
You may have multiple extensions in your `setup.py`.

```python
setup(
    ...
    build_golang={'root': 'github.com/user/project'},
    ext_modules=[Extension('example', ['example.go'])],
    setup_requires=['setuptools-golang'],
    ...
)
```

## Writing cpython extensions in golang

Here's some [examples](https://github.com/asottile/setuptools-golang-examples)

## Common issues

### ```undefined reference to `some_c_function'```

`Extension` by default will bring along the go files listed, but won't bring
along the related C files.  Add the following to `MANIFEST.in`:

```
global-include *.c
global-include *.go
```

### `fatal: could not read Username for 'https://github.com':`

You're probably trying to import from an external source which does not exist.
Double check that your import is correct.


### `package github.com/a/b/c: /tmp/.../github.com/a/b exists but /tmp/.../github.com/a/b/.git does not - stale checkout?`

You've probably mistyped an import.  Double check that your import is correct.

### `duplicate symbol _XXX in: _cgo_export.o mod.cgo2.o`

For example:
```
# github.com/asottile/dockerfile/pylib
duplicate symbol _PyDockerfile_GoParseError in:
    $WORK/github.com/asottile/dockerfile/pylib/_obj/_cgo_export.o
    $WORK/github.com/asottile/dockerfile/pylib/_obj/main.cgo2.o
```

Make sure to mark global variables defined in C as `extern`.
[Here's an example PR](https://github.com/asottile/dockerfile/pull/8)

### repeated rebuilds can be slow

setuptools-golang attempts to make builds more repeatable by using a separate
`GOPATH` -- if you'd like to reuse a GOPATH you can set the
`SETUPTOOLS_GOLANG_GOPATH` environment variable:

```console
$ SETUPTOOLS_GOLANG_GOPATH=~/go pip install .
...
```

## Building manylinux wheels

`setuptools-golang` also provides a tool for building
[PEP 513](https://www.python.org/dev/peps/pep-0513/) manylinux1 wheels so your
consumers don't need to have a go compiler installed to use your library.

Simply run `setuptools-golang-build-manylinux-wheels` from your source
directory.  The resulting wheels will end up in `./dist`.

```
$ setuptools-golang-build-manylinux-wheels

...

+ ls /dist -al
total 8092
drwxrwxr-x  2 1000 1000    4096 Feb  1 04:16 .
drwxr-xr-x 41 root root    4096 Feb  1 04:15 ..
-rw-r--r--  1 1000 1000 2063299 Feb  1 04:16 setuptools_golang_examples-0.1.1-cp34-cp34m-manylinux1_x86_64.whl
-rw-r--r--  1 1000 1000 2064862 Feb  1 04:16 setuptools_golang_examples-0.1.1-cp35-cp35m-manylinux1_x86_64.whl
-rw-r--r--  1 1000 1000 2064873 Feb  1 04:16 setuptools_golang_examples-0.1.1-cp36-cp36m-manylinux1_x86_64.whl
-rw-rw-r--  1 1000 1000    4273 Feb  1 04:14 setuptools-golang-examples-0.1.1.tar.gz
*******************************************************************************
Your wheels have been built into ./dist
*******************************************************************************
```
