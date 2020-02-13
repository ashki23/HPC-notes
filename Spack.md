# Install software with Spack
*[Ashkan Mirzaee](https://ashki23.github.io/index.html) - June 2019*

[Spack](https://spack.readthedocs.io/en/latest/) is a package management tool designed to support multiple versions and configurations of software on a wide variety of platforms and environments. Get spack from the [github repository](https://github.com/spack/spack) and install Lmod as your first package:

```bash
git clone https://github.com/spack/spack.git
source spack/share/spack/setup-env.sh # Setup Spack
```

After setup Spack, let's **install** any new package that you want by:

```bash
spack install <pkg_name>
```

You may find a list of things you can install using Spack [here](https://spack.readthedocs.io/en/latest/package_list.html). 

To **load** the instaled package use:

```bash
spack load <pkg name>
```

Always use tab completion to complete packages' name correctly (press tab once) or find all available names (press tab twice). 

To **uninstall** software ues:

```bash
spack uninstall <pkg name>
```

You may use `--force` flag to remove some packages.

### Lmod
[Lmod](https://www.tacc.utexas.edu/research-development/tacc-projects/lmod) is a Lua-based 
module system that easily handles the MODULEPATH Hierarchical problem. To handel this issue, we can use Lmod to install new packages. To install and activate Lmode use:

```bash
source spack/share/spack/setup-env.sh
spack install lmod
```

To use Lmode to install a new package, use:
```
unset MODULEPATH
unset MODULESHOME
source $(spack location -i lmod)/lmod/lmod/init/bash

spack install <pkg_name>
```

---
Copyright 2018-2019, [Ashkan Mirzaee](https://ashki23.github.io/index.html) | Content is available under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) | Sourcecode licensed under [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
