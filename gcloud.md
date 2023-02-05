# Introduction to cloud computing
*[Ashkan Mirzaee](https://ashki23.github.io/index.html)*

Not many years ago around us was full of CDs, DVDs, Flash Drives and
Hard Drives and it was hard to imagine life without them. But now it
seems very hard to see any of those things around us, what happened
then? The answer is Hard Drives moved to clouds\! For free or for a
small fee we had access to numerous storage somewhere outside our
computers that pushed us to upload every thing that we used to download.

But this is not end of the story and HDDs are not the last part of our
computers that could fly\! Now, CPUs start to move to clouds and this is
beginning of the cloud computing journey. Following are some definitions
of cloud computing:

“Cloud computing is the delivery of computing services—including
servers, storage, databases, networking, software, analytics, and
intelligence—over the Internet (“the cloud”) to offer faster innovation,
flexible resources, and economies of scale" (from
[Azure](https://azure.microsoft.com/en-us/overview/what-is-cloud-computing/)
website).

“Cloud computing is the on-demand delivery of IT resources over the
Internet with pay-as-you-go pricing. Instead of buying, owning, and
maintaining physical data centers and servers, you can access technology
services, such as computing power, storage, and databases, on an
as-needed basis from a cloud provider” (from
[AWS](https://aws.amazon.com/what-is-cloud-computing/) website).

In this tutorial we will learn how to create and access to a compute
engine on [Google Cloud](https://cloud.google.com) and build a Virtual
Machine (VM) that could be accessible from anywhere.

-----

## Google Cloud

To begin, we need a Google Cloud account. Note that Google Cloud
basically is not a free service but you can access some services with no
cost. Follow the instructions in
[here](https://cloud.google.com/compute/docs/quickstart-linux) to create
a new compute engine (VM instance), for example `f1-micro (1 vCPU, 0.6
GB memory, 10GB size)` can be used with no cost. There are several OS
options, but in tutorial we will use [CentOS](https://www.centos.org).

After creating the virtual machine (VM), click on the instance name and
press edit button on the top of the page and add your SSH public key to
the instance. After adding the key, you will see the `username` in the
first column. If you don’t have a public key, open a Linux terminal in
your computer and run:

``` bash
cd ~
ssh-keygen
```

Make sure to create a strong passphrase and remember it for the latter.
To see the public key, use the following in the terminal prompt:

``` bash
cat .ssh/id_rsa.pub
```

Copy the outputs and add it to the instance. Now to connect the VM
instance, we only need to copy the external IP of the instance and run
the following in a Linux terminal prompt and enter the public key
passphrase (find more details
[here](https://cloud.google.com/compute/docs/instances/connecting-advanced#thirdpartytools)):

``` bash
ssh username@external-ip
```

After connecting to the instance, we can change the ssh configuration in
a way that connecting to the VM without a public key. To connect to the
instance with a root password, open the `/etc/ssh/sshd_config` config
file with a text editor (e.g. `sudo emacs /etc/ssh/sshd_config`) and
comment `#PasswordAuthentication no` and uncomment
`PasswordAuthentication yes`. After save and exit, run the following to
apply changes:

``` bash
sudo systemctl restart sshd
```

**Note:** in a Debian based Linux, use `sudo service ssh restart`.

After these changes, we can connect to the instance by both public key
passphrase (if a public key exist) or root password of the instance (if
there is no public key). Note that the VM’s in cloud engine do not come
with a root password setup by default, use the following to set it up:

``` bash
sudo passwd user
```

To find the `user` name run `whoami` command (that should be same as the
`username` pair to the public key).

Also, we can use *Chrome Secure Shell App* by entering the username and
external IP address of the instance such that:

``` bash
username@external-ip
```

We can use root password to connect (because of changes in the SSH
config file) but if you want to use the public key, select the related
*private SSH key* in the **Identity** field. If necessary, click Import
to select a private key file from your local workstation
(`~/.ssh/id_rsa`) - if using macOS, press `shift+command+.` to see
hidden files.

Here we go\! now we can connect from a terminal prompt of a local
computer that has the public key, or from a Chrome browser from any
computer. Note that if multiple users are going to connect to the
instance, it will be better to add a [OS
login](https://cloud.google.com/compute/docs/instances/managing-instance-access)
to the instance by adding a metadata entry where the key is
`enable-oslogin` and the value is `TRUE`.

## CentOS

Connecting to the instance, connect us to a computer with CentOS
operating system. In the following we will learn how to update CentOS
and install required software.

If you have some experiences with Linux, probably you are familiar with
Debian package manger (`apt`). CentOS uses a different package manager
called `yum` which is pretty similar to `apt`. We can check for the
update by:

``` bash
sudo yum check-update
```

And update OS by:

``` bash
sudo yum update
sudo yum clean all # clean all cache files
```

**Note:** we can find OS version by `cat /etc/redhat-release`.

To install, update or remove packages use:

``` bash
sudo yum install pkg
sudo yum update pkg
sudo yum remove pkg
```

Let’s install `wget`, `git` and `emacs` by:

``` bash
sudo yum install wget # to download files from the Internet
sudo yum install git # to access Git
sudo yum install emacs # to have Emacs text editor
```

To install [Miniconda3](https://docs.conda.io/en/latest/miniconda.html)
run:

``` bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

Lets refresh the `.bashrc` file to add `conda` command to the current
environment by:

``` bash
source ~/.bashrc
```

If you’d prefer that conda’s `base` environment not be activated on
startup, run:

``` bash
conda config --set auto_activate_base false
```

And use the following to activate conda `base` later:

``` bash
conda activate base
```

See
[here](https://conda.io/projects/conda/en/latest/user-guide/install/linux.html)
for more details about installing Miniconda and learn more about using
conda in
[here](https://docs.conda.io/projects/conda/en/latest/user-guide/getting-started.html).
Also, review a tutorial about conda in
[here](https://ashki23.github.io/python_env.html#miniconda).

**Note:** (optional) in CentOS `python` command refers to Python 2. To
access `python3` command, we can add `alias
python3=~/miniconda3/bin/python3` to `~\.bashrc` file. Use `source
~/.bashrc` to apply changes.

To install R, we can use conda to create an R environment:

``` bash
conda create --prefix ./r_env r-base
```

To use R, first activate the `r_env` by `conda activate ./r_env` (when
you are in the same directory as `r_env`) and run `R`.

Now we can download and install other required software. `yum` and
`conda` package managers can be used to install different software and
packages same as what we did in above to install Emacs and R. Also, we
can directly install software by downloading source files and install
them such as installing Miniconda in above. After installation, we can
reboot the engine by:

``` bash
sudo reboot
```

Now I think your VM is ready, enjoy it\!

---

Copyright 2018-2023, [Ashkan Mirzaee](https://ashki23.github.io/index.html) | Content is available under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) | Sourcecode licensed under [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
