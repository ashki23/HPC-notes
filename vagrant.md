# Create and manage virtual machines with Vagrant
*[Ashkan Mirzaee](https://ashki23.github.io/index.html)*

[Vagrant](https://www.vagrantup.com/intro) is a tool for building and
managing virtual machine environments in a single workflow. With an
easy-to-use workflow and focus on automation, Vagrant lowers development
environment setup time, increases production parity, and provides an
end-to-end VM management tool on your terminal.

  - [Documentation](https://www.vagrantup.com/docs)
  - [Quick
    Start](https://learn.hashicorp.com/tutorials/vagrant/getting-started-index)

This document shows how to install Vagrant and create virtual machines
(VM).

-----

## Install

[Download](https://www.vagrantup.com/downloads) Vigrant and unzip the
file in the location the you want i.e. `/home` or `/opt`. To use
Vagrant, we need [VirtualBox](https://www.virtualbox.org/) to be
installed first. In Linux (Ubuntu) terminal we can use:

``` bash
sudo apt install virtualbox
wget https://releases.hashicorp.com/vagrant/2.2.10/vagrant_2.2.10_linux_amd64.zip
unzip vagrant_*_linux_amd64.zip -d ~/vagrant # we used home directory here
echo "alias vagrant='~/vagrant/vagrant'" >> ~/.bashrc # add vagrant command to terminal
```

Make sure to update 2.2.10 with the latest Vagrant version. Note that
installing VirtualBox required root access but using it doesn’t need
that privilege.

## Boxes

Boxes are the package format for Vagrant environments. These are
actually OS images that we need to create a VM. Vagrant boxes are
available in [Vagrant Cloud](https://app.vagrantup.com/boxes/search). To
create a VM, we need to know the box USER/NAME or URL. For instance,
`hashicorp/bionic64` is a basic Ubuntu 18.04 64-bit box that published
by HashiCorp (the makers of Vagrant).

## First VM

To create your first VM, go to the directory that you want to create the
VM and enter:

``` bash
cd ./path-to/my-first-vm
vagrant init hashicorp/bionic64
```

To setup a basic Ubuntu 18.04 (Bionic) 64-bit VM. Start the VM by:

``` bash
vagrant up
```

And login to the VM by:

``` bash
vagrant ssh
```

Note that `vagrant ssh` is the same as `ssh -p <port>
vagrant@localhost`. Where port can be found by `vagrant port` command
and the password is “vagrant”. By default, username and password are
“vagrant” but we can add other users with new passwords after login to
the VM by:

``` bash
sudo adduser <new-username>
```

We can logout by `logout` command or `Ctrl + D`. Also, we can suspend or
resume a VM by `vagrant suspend` and `vagrand resume` commands. And we
can stops and deletes all traces of the vagrant machine by `vagrant
destroy` command. Finally, we can start the VM again by `vagrant up`.

## Commands

Vagrant has set of commands to run and manage VMs. Use `vagrant --help`
to see list of the common commands and their description. Each of these
command has some subcommand that can be found by `vagrant --help
<command>` for example `vagrant box --help` shows available subcommands
and options for box command.

``` bash
vagrant init USER/NMAE # initialize a new vagrant env by creating a Vagrantfile
vagrant up # start/restrat and provision the vm
vagrant halt # stop the vm
vagrant destroy # stop and delete the vm
vagrant provision # provision the vm
vagrant reload # restart the vm and load new Vagrantfile configuration
vagrant suspend/resume # suspend/resume the vm
vagrant status # status of the vm
vagrant ssh # connect to the vm via ssh
vagrant port # info about guest port mappings
vagrant box add USER/NAME # add a box without initializing (without creating a Vagrantfile)
vagrant box list # show list of boxes
vagrant box outdated --global # check boxes are update
vagrant box remove USER/NAME # remove a box
```

## Vagrantfile

[Vagrantfile](https://www.vagrantup.com/docs/vagrantfile) is a text file
with [Ruby](https://www.ruby-lang.org/en/) syntax, which has all the
information about configuring and provisioning a set of machines. When
you initiate a VM by `vagrant init hashicorp/bionic64`, a Vagrantfile is
creating by default such that:

``` ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "hashicorp/bionic64"
  
  # Change the hostname from vagrant.
  # config.vm.hostname = "new-hostname"
  
  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # Note: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory and cpu on the VM:
  #   vb.memory = "1024"
  #   vb.cpus = "2"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
```

We can modify the Vagaratfile and reload the VM by `vagrant reload` or
create our template file including all configuration that are needed. If
a Vagrantfile is available, We can start the VM by running `vagrant up`
command.

In general there are three types of Vagrant configs:

  - [`config.vm`](https://www.vagrantup.com/docs/vagrantfile/machine_settings):
    modify the configuration of the machine that Vagrant manages
  - [`config.ssh`](https://www.vagrantup.com/docs/vagrantfile/ssh_settings):
    relate to configuring how Vagrant will access your machine over SSH
  - [`config.vagrant`](https://www.vagrantup.com/docs/vagrantfile/vagrant_settings):
    modify the behavior of Vagrant itself

---

Copyright 2018-2021, [Ashkan Mirzaee](https://ashki23.github.io/index.html) | Content is available under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) | Sourcecode licensed under [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
