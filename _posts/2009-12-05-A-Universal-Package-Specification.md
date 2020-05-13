---
layout: post
excerpt: |
  One of my favourite things about Unix-like operating systems is
  package management. I discuss the Archlinux PKGBUILD format and
  propose a new metadata format for packages.
categories: [tech]
---
One of my favourite things about Unix-like operating systems is package
management. They have many technical advantages such as:

 * ease of installation - just search, install,
 * automatic dependency resolution,
 * reduction in storage usage by utilizing shared libraries (no [DLL hell]),
 * clean filesystem,
 * etc.

It also has many practical advantages such as listing installed packages,
showing package information (version, url), and finding which package a file
belongs to. Unfortunately the implementation of this concept isn't always
perfect. For instance, the learning curve for [Debian packaging] is quite
steep, and RPM has often been criticised for [dependency hell].

When I first looked at [Archlinux]'s package management system, I was
pleasantly surprised at how simple the [PKGBUILD] format is. It is a shell
script, in which variables define metadata, and a `build()` function
specifies the steps required to build the package.

## Problems

The [Archlinux User Repository][AUR] allows users to contribute PKGBUILDs
without a lengthy review process. The code behind it wasn't in the greatest
shape, so I decide to rewrite it in Python + Django. During development I ran
into the problem of parsing the PKGBUILD format. I had to get the metadata
into the database somehow. Since a PKGBUILD is just a shell script, I thought
sourcing it and outputting Python, then eval-ing it would be the easiest way
of doing it. Unfortunately this has many security problems, including the
execution of malicious code, or infinite loops (the web server would hang). I
was forced to write a minimal [shell parser][pkgparse] to extract the
metadata. While it removes the security concerns, it raises others, such as
inaccuracy and maintenance problems (the specification and parser code is
tightly coupled). It turns out the [Shell grammar] isn't exactly the simplest
one.

The PKGBUILD format has some warts, which are mostly due to the lack of data
structures in bash, such as hash tables. For instance there are two arrays,
`source` and `md5sums`. Each element in the `md5sums` array maps to the
respective `source` element. As you can imagine this can easily result in
bugs. Fortunately `makepkg` (the package creation utility) is able to create
this field automatically.

Another problem associated with `sources` and checksums is with binary
sources. A binary source is typically different for each supported
architecture. There is no way of specifying this easily in the PKGBUILD
format. A common solution is to do something like this:

```bash
arch=('i686' 'x86_64')
if [ "$CARCH" = "x86_64" ]; then
	source=("http://foo.com/${pkgname}-${pkgver}-x86_64.bin")
	md5sums=(4dacc4474e93bcd4e168fdc48c4e6aee)
else
	source=("http://foo.com/${pkgname}-${pkgver}-i386.bin")
	md5sums=(5001378e4f83d0d6db014eec9182f7b4)
fi
```

The checksum generation feature of `makepkg` no longer works properly, and in
order to parse this metadata, an interpreter is now required, not just a
parser.

## Solutions

An Archlinux user started defining [an alternate PKGBUILD
specification][pkgmeta]. It addresses some problems with the shell format,
such as extendability (to a degree) and ease of parsing. Unfortunately this
format is a completely new data format, and thus requires a parser of its
own.

Lately I've been toying with the idea of creating a universal package
specification. The idea of this specification is to provide a portable
way of defining package metadata, while keeping it simple, and extendable.
Ideally any package manager would be able to use this format, and have enough
metadata to do what they need to. It is extendable with an `extensions`
field, which allows package managers to get any data they require, which is
not already included in the specification. If there is enough demand for an
extension, it should be added to the next revision of the specification.

A common data serialization format is [YAML]. It is simple, easy to parse, and
very versatile. For these reasons it was my first choice for the
specification. There are already many different parsers in many different
languages. Thus, the format should be easily accessible in most languages.
Unfortunately it does not seem that bash is one of them, so parts of
`makepkg` would have to be rewritten.

## Suggested Format

```yaml
name: foo
version: 1.0
release: 1
summary: A fictional package to show the YAML package format
description: >
  This package is an example of the YAML universal package format, which
  aims to be portable and extandable. This description can be as long as
  it needs to be.
architectures: # Keys denote architectures supported by this package
  any:
    sources: # URIs of source files, such as a source tarball
      - uri: http://www.foo.org/{{name}}-{{version}}.tar.gz
        checksums: # keys denote algorithm, values are the checksums
          md5: fd085a845298afb36f6676feac855e63
          sha1: e19f73b340aeae43b98908006094af62e0c7b5b9
      - uri: shared_data-{{version}}.tar.gz
        checksums:
          md5: 2edd3a155dcbb6632029accd2926d33b
          sha1: 776b13c7ff8e8b120a1ea4910a8b8b64c289e6b6
        extract: Yes # Whether an archive should be extracted
icon: foo.png # An icon for GUI package managers
licenses: [MIT]
url: http://www.foo.org
categories: [fictional, example] # Keywords associated with this package
distributions: null # Allows to install a group of packages as a single
                    # target
requires:
  - bar >= 2.1
  - eggs == 1.1.2
provides:
  - example
conflicts:
  - foobar
optional: # Keys denote optional packages, values describe why they are
          # optional and what features they provide
  - eggs: To make a nice omelette
  - spam: If you like that sort of thing...
extensions: # User-defined data
  # The following is metadata used by makepkg, but not used by other
  # common package managers.
  options: [trip, "!docs", libtool, emptydirs, zipman]
  backup:
    - /etc/foo.rc
  install: foo.install
  build: |
    ./configure --prefix=/usr
    make
    make DESTDIR=$pkgdir install
#...
```

Most of these fields are analogous to those in the current PKGBUILD format.
The major difference is with `architectures`. The keys of this hash table
denote supported architectures. The *any* architecture has two sources. The
URI and checksums are defined. No longer is a conditional required. The
appropriate architecture is simple retrieved and its sources are used. The
source URLs and checksums now have a one-to-one mapping, reducing human
error. There is still one problem with this, however. If the sources do not
differ for multiple architectures, there will be duplication. To rectify this
to some degree, anchors can be used:

```yaml
architectures:
  i686: &common_sources
    sources:
      - uri: http://www.foo.org/{{name}}-{{version}}.tar.gz
        checksums:
          md5: fd085a845298afb36f6676feac855e63
          sha1: e19f73b340aeae43b98908006094af62e0c7b5b9
  x86_64: *common_sources
```

In some cases it might even be useful to exploit hash merges to specify a
common subset of sources and add architecture-specific ones.

Another re-use of data was common in PKGBUILDs - using variables to reference
the package name and version in sources. It looked something like this:

```bash
pkgname="foo"
pkgver=1.0
source=("http://foo.org/${pkgname}-${pkgver}.tar.gz")
```

You might have noticed that a similar syntax was used in the uri:

```bash
- uri: "http://www.foo.org/${name}-${version}.tar.gz"
```

This is not something that YAML supports - it would have to be parsed
separately. I have not decided on this format yet, and perhaps YAML does
indeed have an appropriate feature. For now these values can either be
hardcoded, or parsed on the second pass of the data.

The `extensions` field is the interesting part. If the specification doesn't
have some required data, such as the `options` field in the PKGBUILD
specification, it can be added here.

[dll hell]: http://en.wikipedia.org/wiki/DLL_hell/
[debian packaging]: http://www.debian.org/doc/debian-policy/
[dependency hell]: http://en.wikipedia.org/wiki/Dependency_hell/
[archlinux]: http://www.archlinux.org/
[PKGBUILD]: http://www.archlinux.org/pacman/PKGBUILD.5.html
[AUR]: http://aur.archlinux.org/
[pkgparse]: http://www.github.com/sebnow/pkgparse/
[Shell grammar]: http://www.opengroup.org/onlinepubs/009695399/utilities/xcu_chap02.html#tag_02_10
[pkgmeta]: http://xyne.archlinux.ca/ideas/pkgmeta/
[YAML]: http://www.yaml.org/
