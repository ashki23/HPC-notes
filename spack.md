# Install software with Spack
*[Ashkan Mirzaee](https://ashki23.github.io/index.html)*

[Spack](https://spack.readthedocs.io/en/latest/) is an open source
package manager that simplifies building, installing, customizing, and
sharing HPC software stacks. Spack helps installing software with much
less efforts. It is a very simple and efficient way for installing
packages with cumbersome structures and lots of dependencies. Spack is
an open source package manger and developing and maitaining by community
of HPC developers.

To learn more about Sapck review:

  - [Tutorial:
    Spack 101](https://spack-tutorial.readthedocs.io/en/latest/)
  - [Spack Basic
    Usage](https://spack.readthedocs.io/en/latest/basic_usage.html)

-----

## Usage

Spack has a very simple workflow. Basically, it is a Git repo of package
files are written in pure Python including software build requirements
and dependencies. We can Clone and setup Spack by:

``` bash
git clone https://github.com/spack/spack.git
source spack/share/spack/setup-env.sh # setup Spack
```

To list a specific software and it’s specifications we can use:

``` bash
spack list <software-name> # list a software
spack info <software-name> # show information and variants of a software
spack spec -I <software-name> # show specifications (dependencies)
```

Option `-I` or `--install-status` shows status of the software
dependencies i.e. installed (`+`) or will be installed during the
installation (`-`).

Now let’s install a new softwares from Spack:

``` bash
spack install <software_name> or <software_name@version> or <software_name@version %compiler@version> 
```

In general, `@version` for both software and compiler could be removed.
Spack installs the most stable version by default (see `spack versions
-s <software-name>`). You may find complete list of software that you
can install by using `spack list` or in Spack online [package
list](https://spack.readthedocs.io/en/latest/package_list.html). Also.
we can use `--verbose` option to see more details during the
installation, `--no-cache` to install a package directly from the
source, and `--overwrite` to overwrite an installed package.

After installation, we can find the software by:

``` bash
spack find # to see all installed software
spack find <software-name> # to find a software (use -lfvp to see hashes, flags, variants and pathes)
spack location -i <software-name> # to find location of a software 
```

And we can load the software:

``` bash
spack load <software_name>
```

To uninstall a software we can use:

``` bash
spack uninstall <software_name> or <software_name@version> or <software_name@version %compiler@version>
```

We can use `-R` or `--dependents` option to uninstall depndencies and
`--force` to force to remove.

More details about Spack commands can be found by:

  - `spack help` getting help
  - `spack help subcommand` print out usage information for a particular
    subcommand (eg. `spack help find`)

To see full list of commands visit Spack [Command
Reference](https://spack.readthedocs.io/en/latest/command_index.html).

## Specs & dependencies

In general, we can specify versions by `@`, compilers by `%`,
dependencies by `^` and hashes by `/` for install/uninstall commands.
The following example from [Spack Basic
Usage](https://spack.readthedocs.io/en/latest/basic_usage.html?highlight=cflags#specs-dependencies)
show these specs:

``` bash
mpileaks@1.2:1.4 %gcc@4.7.5 +debug ~qt arch=bgq_os ^callpath@1.1 %gcc@4.7.2
```

If provided to `spack install`, this will install the `mpileaks` library
at some version between 1.2 and 1.4 (inclusive), built using `gcc` at
version 4.7.5 for the `Blue Gene/Q` architecture, with `debug` options
enabled, and without `Qt` support. Additionally, it says to link it with
the `callpath` library (which it depends on), and to build `callpath`
with `gcc` 4.7.2.

## Compilers

We can select compiler version and settings. To find and list compilers,
use:

``` bash
spack compiler list
spack compiler find
```

Compilers can be added to the Spack compilers list or removed from the
list by:

``` bash
spack compiler add <compiler-name@ver> # for example $(spack location -i gcc@10.1.0) add gcc 10 compiler that already is installed by Spack
spack compiler remove <compiler-name@ver>
```

Also, we can directly modify `compilers.yaml` file by:

``` bash
spack config edit compilers
```

Moreover, we can add specific compiler’s options during the
installation. Valid flag names are `cflags`, `cxxflags`, `fflags`,
`cppflags`, `ldflags`, and `ldlibs`. For instance, `spack install
libdwarf cppflags="-g"` will install `libdwarf` with the `-g` flag
injected into their compile line.

## Architecture specifiers

Also, we can specify architecture of a machine by `arch`. For example,
the following will unistall all software installed with `gcc` 4.8.5 in
`linux-centos7-haswell` machine:

``` bash
spack uninstall --all %gcc@4.8.5 arch=linux-centos7-haswell
```

We can specify these values separately by `platform=linux`, `os=centos7`
and `target=haswell`.

## Create and edit package files

If the package that you are looking for is not among Spack repo list,
you can create a package file in Python by `spack create <tarball link>`
and [add to the Spack
reop](https://spack.readthedocs.io/en/latest/contribution_guide.html).
Review [Package Creation
Tutorial](https://spack-tutorial.readthedocs.io/en/latest/tutorial_packaging.html)
to learn how to build and debug a package.

If the package that you are looking for is outdated or has wrong
dependencies, you can modify the package Python file by:

``` bash
spack edit <software-name>
```

To update version of the package, install the package tar file from the
developer webpage or Github and extract the related hash number by
`md5sum` or `shasum -a 256` command and add version and hash number to
the package file. For example, `spack edit octopus`:

``` bash
class Octopus(Package):
    """A real-space finite-difference (time-dependent) density-functional
    theory code."""

    homepage = "https://octopus-code.org/"
    url      = "http://octopus-code.org/down.php?file=10.0/octopus-10.0.tar.gz"

    version('10.0',   sha256='ccf62200e3f37911bfff6d127ebe74220996e9c09383a10b1420c81d931dcf23')
```

In the above example, we have added Octopus version 10 to the package
file.

Also, we can open the packages configuration file and update preferences
by:

``` bash
spack config edit packages
```

**Note:** if you prefere to use Emacs to edit Spack config files, use
`export EDITOR=emacs` to make Emacs the default text editor.

## Lmod

[Lmod](https://www.tacc.utexas.edu/research-development/tacc-projects/lmod)
is a Lua-based module system that easily handles the MODULEPATH
Hierarchical problem. To install Lmode, use the following (note this may
take a while):

``` bash
source spack/share/spack/setup-env.sh
spack install lmod
```

To use Lmode for installing and loading new packages use:

``` bash
unset MODULEPATH
unset MODULESHOME
source spack/share/spack/setup-env.sh
source $(spack location -i lmod)/lmod/lmod/init/bash

spack install <software-name>
module load <software-name>
```

Learn more about Lmod and module files in
[here](https://spack-tutorial.readthedocs.io/en/latest/tutorial_modules.html).

---

Copyright 2018-2019, [Ashkan Mirzaee](https://ashki23.github.io/index.html) | Content is available under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) | Sourcecode licensed under [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
