# Python, Openstack, and You

A guide to the Python and Openstack development ecosystems. (WIP!)

## Build

This project uses [pandoc](http://johnmacfarlane.net/pandoc/) for
generating the guide. You can either use your distribution's package
manager to install pandoc directly, or you can install
[haskell-platform](http://www.haskell.org/platform/) and then:

```bash
$ cabal update
$ cabal install pandoc
```

### PDF

If you have a LaTeX environment set up, you can do the following to produce a PDF:

```bash
$ cd src
$ pandoc --toc guide.md -o guide.pdf
```

The ```---toc``` flag generates a table of contents. This may be
omitted, as the Table of Contents is still embedded in the metadata
for the guide, thanks to pandoc magic.

### Other Formats

pandoc supports an amazing number of
[formats](http://johnmacfarlane.net/pandoc/). The build process is
generally:

```bash
$ pandoc --toc guide.md -o guide.<format>
```

## License

![](http://i.creativecommons.org/l/by-sa/4.0/88x31.png)

Python, Openstack, and You is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/deed.en_US).
