# Install software with Spack
*[Ashkan Mirzaee](https://ashki23.github.io/index.html)*

[Spack](https://spack.readthedocs.io/en/latest/) is an open source
package manager that simplifies building, installing, customizing, and
sharing HPC software stacks. Spack helps installing software with much
less efforts. It is a very simple and efficient way for installing
packages with cumbersome structures and lots of dependencies. Spack is
an open source package manger and developing and maintaining by
community of HPC developers.

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

Now let’s install a new software’s from Spack:

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
spack find --loaded # see what is loaded
```

To uninstall a software we can use:

``` bash
spack uninstall <software_name> or <software_name@version> or <software_name@version %compiler@version>
```

We can use `-R` or `--dependents` option to uninstall dependencies and
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
developer web-page or GitHub and extract the related hash number by
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

**Note:** if you prefer to use Emacs to edit Spack config files, use
`export EDITOR=emacs` to make Emacs the default text editor.

## Environment modules

[Lmod](https://www.tacc.utexas.edu/research-development/tacc-projects/lmod)
is a Lua-based module system that easily handles the `MODULEPATH`
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
source $(spack location -i lmod)/lmod/lmod/init/bash
source spack/share/spack/setup-env.sh

spack install <software-name>
module load <software-name>
```

Now we can use `module avail` to see list of the available (installed)
modules.

We can also, use `lomd` to manage modules by editing the module
configuration file. Let’s open the file:

``` bash
spack config edit modules 
```

And add the following configuration from [Spack
tutorial](https://spack-tutorial.readthedocs.io/en/latest/tutorial_modules.html#modules-tutorial)
to the config file:

``` bash
modules:
  enable::
    - lmod
  lmod:
    core_compilers:
      - 'gcc@7.5.0'
    hierarchy:
      - mpi
      - lapack
    hash_length: 0
    whitelist:
      - gcc
    blacklist:
      - '%gcc@3.8.0' 
      - readline
    all:
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
      environment:
        set:
          '{name}_ROOT': '{prefix}'
    openmpi:
      environment:
        set:
          SLURM_MPI_TYPE: pmi2
          OMPI_MCA_btl_openib_warn_default_gid_prefix: '0'
    netlib-scalapack:
      template: 'group-restricted.lua'
```

Note that in the above example, we seleceted `gcc@7.5.0` as core
compiler and hided `gcc@3.8.0` from the tree. Also, we supposed the
hierarchy of installed modules are depends on `mpi` and `lapack`.

To apply these changes, we need to refresh the modules tree and update
`MODULEPATH` by:

``` bash
module purge
spack module lmod refresh --delete-tree -y
module unuse $HOME/spack/share/spack/modules/<os-arch>
module use $HOME/spack/share/spack/lmod/<os-arch>/Core
```

Note that in above commands, I assumed that Spack directory is located
on `$HOME`. You need to update path and `<os-arch>` based on the Sapck
directory and, the operating system and architecture of your machine
(for example `linux-ubuntu18.04-x86_64`). To keep the new module path
for latter, you can add the following to the `.bashrc` file.

``` bash
export MODULEPATH=$HOME/spack/share/spack/lmod/<os-arch>/Core
```

Now use `module avail` to see changes. You can learn more about modules
and Lmod in
[here](https://spack-tutorial.readthedocs.io/en/latest/tutorial_modules.html).

---

Copyright 2018-2019, [Ashkan Mirzaee](https://ashki23.github.io/index.html) | Content is available under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) | Sourcecode licensed under [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
