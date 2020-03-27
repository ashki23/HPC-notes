# Install software with Spack
*[Ashkan Mirzaee](https://ashki23.github.io/index.html) - June 2019*

[Spack](https://spack.readthedocs.io/en/latest/) is a software/package management tool designed to support multiple versions and configurations of software on a wide variety of platforms and environments. Let's download and install Spack by:

```bash
git clone https://github.com/spack/spack.git
source spack/share/spack/setup-env.sh # Setup Spack
```

Now we can easiliy **install** new softwares from Spack by:

```bash
spack install <software_name> or <software_name@version> or <software_name@version %compiler@version> 
```

In general, `@version` for both softwares and compilers could be removed, Spack installs the most stable version by default (see `spack versions -s software_name`). You may find complete list of things that you can install by using `spack list` or online in [here](https://spack.readthedocs.io/en/latest/package_list.html). 

To use the software, first we need to **load** it by:

```bash
spack load <software_name>
```

Always use tab completion to complete softwares' name correctly (press tab once) or find all available names (press tab twice). 

To **uninstall** a software and every package that depends on it, you may give the `--dependents` option such that:

```bash
spack uninstall --dependents <software_name> 
```

You may use `--force` flag to remove some packages.

Some other important Spack commands are:

- `spack list` to list all avail sofeware
- `spack list name` to list all avail software contain that name
- `spack info software_name` to get more information on a particular package
- `spack versions -s software_name` to see all safe version of particular package
- `spack find` to show list of all installed software by Spack
- `spack find name` to find all installed software contain that name
- `spack location -i software_name` to find install prefix
- `spack gc` use gc ("garbage collector") to uninstall all unneeded packages
- `spack help` getting help
- `spack help subcommand` print out usage information for a particular subcommand (eg. `spack help find`)

To learn more, review Spack Basic Usage in [here](https://spack.readthedocs.io/en/latest/basic_usage.html).

### Lmod
[Lmod](https://www.tacc.utexas.edu/research-development/tacc-projects/lmod) is a Lua-based 
module system that easily handles the MODULEPATH Hierarchical problem. To handel this issue, we can use Lmod to install new packages. To install Lmode, use the following (note this may take a while):

```bash
source spack/share/spack/setup-env.sh
spack install lmod
```

To use Lmode for installing and loading new packages use:
```bash
unset MODULEPATH
unset MODULESHOME
source $(spack location -i lmod)/lmod/lmod/init/bash
source spack/share/spack/setup-env.sh

spack install <software_name>
module load <software_name>
```

Find more information about modules in [here](https://spack-tutorial.readthedocs.io/en/latest/tutorial_modules.html) and learn the workflow to make a Spack package in [here](https://spack.readthedocs.io/en/latest/workflows.html).  

---
Copyright 2018-2019, [Ashkan Mirzaee](https://ashki23.github.io/index.html) | Content is available under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) | Sourcecode licensed under [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
