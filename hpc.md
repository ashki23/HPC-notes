# Introduction to high-performance computing (HPC)
*[Ashkan Mirzaee](https://ashki23.github.io/index.html)*

When dealing with computational problems, various resource and time
limitations could arise difficulties and disrupt the solutions. If we
have enough time and resources, we might find the answer. A
supercomputer or cluster with high level of performance could help us
tackle the problem. The followings are some great workshops about HPC:

  - [Introduction to High-Performance
    Computing](https://hpc-carpentry.github.io/hpc-intro/)
  - [Introduction to using the shell in a High-Performance Computing
    context](https://hpc-carpentry.github.io/hpc-shell/)

Using HPC systems often involves the use of a Shell through a command
line interface which is necessary for this topic (see
[here](https://ashki23.github.io/shell.html)).

This tutorial provides a list of basic scheduling commands, submitting
jobs, methods of transferring files from local computers, and installing
software on clusters.

Related document:

  - [Introduction to using
    clusters](https://ashki23.github.io/cluster.html)

-----

## Scheduling jobs

On an HPC system, we need a scheduler to manage how jobs running on a
cluster. One of the most common schedulers is
[SLURM](https://slurm.schedmd.com/). The following are some practical
SLURM commands ([quick start user
guide](https://slurm.schedmd.com/quickstart.html)):

``` bash
sinfo -s # shows summary info about all patritions
sjstat -c # shows computing resources info
srun # run parallel jobs
sbatch # submit a job to the scheduler
JOB_ID=$(sbatch --parsable file.sh) # keep the JOB ID right after reading the command
sbatch --dependency=afterok:JOB_ID file.sh # submit a job file after finishing other jobs
sbatch --dependency=singleton # submit a job after ending a job with a same name 
sacct # displays accounting data for all jobs and job steps in the SLURM job accounting log
squeue -u <userid> # check on a user's job status
squeue -u <userid> --start # show estimation time to start pending jobs
scancel JOBID # cancel the job with JOBID
scancel -u <useride> # cancel all the user jobs
```

To see more details about these commands use `<command> --help`.

Let’s connect to the cluster through `ssh user@server`, and do some
practices. For example, use `nano example-job.sh` to make a job file
including:

``` bash
#!/bin/bash
#SBATCH --nodes 1
#SBATCH --mem 16G
#SBATCH --ntasks 1
#SBATCH --cpus-per-task 4
#SBATCH --partition hpc0
#SBATCH --account general
#SBATCH --time 02-05:00
#SBATCH --job-name NewJobName
#SBATCH --mail-user your@email.com
#SBATCH --mail-type END

echo 'This script is running on:'
hostname
sleep 120
```

Special characters `#!` (shebang) at the beginning of scripts specifies
what program should be used (i.e. `/bin/bash` or `/usr/bin/python3`).
SLURM uses `#SBATCH` special comment to denote special
scheduler-specific options. To see more options, use `sbatch --help`.
For example, the above file uses 1 nodes, 16 gigabytes of memory, 1 taks
and 4 CPUs per task, and using partition `hpc0` with a `general` account
for 2 days and 5 hours of walltime, gives a new name to the job, and
email you when the job is ended. Now we can submit the job file by
`sbatch example-job.sh`. We can use `squeue -u USER` or `sacct` to check
the job file status, and use `scancel JOBID` to cancel the job. You may
find more sbatch options [here](https://slurm.schedmd.com/sbatch.html).

To run a single command, we can use `srun`. For instance, `srun -c 2
echo "This job will use 2 CPUs."` submits a job and allocates 2 CPUs.
Also, we can use `srun` to open a program in an interaction mode. For
example, `srun --pty bash` will open a Bash shell in a computation node
(not specified).

**Note**: in general, when we connect to a cluster we will go to a node,
called **login node**, which is not meant to do heavy computational
tasks. So, to do our computations in a proper way, we should always use
either **`sbatch`** or **`srun`**.

Usually there are many **modules** available on the clusters. To find
and load these modules use:

``` bash
module avail # shows all avaliable madules (programs) in the cluster
module load <name> # to load a module ex. module load R or python
module list # shows list of the loaded modules
module unload <name> # to unload a module
module purge # to unload all modules
```

To create a simple template `sbatch` job file, use the following steps:

1.  generate any files including all codes that we want to run in the
    cluster (that could be several Python or R or other scripts)
2.  generate a Bash file including all modules that are required for the
    sending job (`environment.sh`)
3.  generate a Bash file to call steps 1 and 2 including all `#SBATCH`
    options (`job_file.sh`)
4.  use `sbatch`to run the file in step 3

For example, let’s run the following Python code called `test.py`:

``` bash
#!/usr/bin/python3

print("Hello world")
```

Then use `nano environment.sh` to create the environment file including:

``` bash
#!/bin/bash

module load miniconda3
```

Then use `nano job-test.sh` to make the job file by:

``` bash
#!/bin/bash

#SBATCH --mem 1G
#SBATCH --job-name Test1

echo === $(date)
echo $SLURM_JOB_ID

source ./environment.sh
module list

srun python3 ./test.py

echo === $(date) $(hostname)
```

Now we can use `sbach job-test.sh` to run this job.

If there are some dependencies between jobs, `slurm` can defer the start
of a job until the specified dependencies have been satisfied completed.
For instance, let’s create another job called `job-test-2.sh`:

``` bash
#!/bin/bash

#SBATCH --mem 1G
#SBATCH --job-name Test2

echo === $(date)
echo $SLURM_JOB_ID
echo === This is a new job
echo === $(date) $(hostname)
```

We need another job, called `job-test-3.sh`, to run both `job-test.sh`
and `job-test-2.sh`:

``` bash
#!/bin/bash

#SBATCH --mem 1G
#SBATCH --job-name Dependency

echo === $(date)
JID=$(sbatch --parsable job-test.sh)
echo $JID

sbatch --dependency=afterok:$JID job-test-2.sh
echo === $(date) $(hostname)
```

Where `JID` is the job ID for `sbatch job-test.sh` which is a dependency
for `job-test-2.sh`. Now, by running `sbatch job-test-3.sh` we will make
sure that `job-test-2.sh` will run after that `job-test.sh` is completed
successfully.

Note that there are some other tools, such as
[Snakemake](https://snakemake.readthedocs.io/en/stable/), that could be
used for workflow management.

-----

## Transferring files

### Secure copy (scp)

We can use secure copy or `scp` to transfer files from a local computer
to a cluster and vice versa. For example, let’s transfer
`code_example.py` from `temp/` directory on the `remote.edu` cluster to
`Documents/` directory in your local computer when we are using the
local computer. For this we can use:

``` bash
cd ~/Documents
scp user@remote.edu:/temp/code_example.py .
```

The `.` at the end of the code says, paste with the same name of the
source file. To do the reverse task with:

``` bash
cd ~/Documents
scp code_example.py user@remote.edu:/temp/.
```

To recursively copy a directory (with all files in the directory), we
just need to add the -r (recursive) flag. For example to download temp
folder use:

``` bash
cd ~/Documents
scp -r user@remote.edu:/temp .
```

### Rsync

Rsync is a fast, versatile, remote (and local) file-copying tool. Rsync
has two great features, first it always syncs your data (i.e. only
transfer files that changed since last transfer) and second, the
compress option that makes transferring large files easier. To use
`rsync`, follow:

``` bash
# From local to remote
rsync local-directory user@remote.edu:remote-directory

# From remote to local
rsync user@remote.edu:remote-directory local-directory
```

That recursively transfer all files from the directory `local-directory`
on the local machine into the `remote-directory` on the remote machine.
Some important options for `rsync` are (use `rsync -help` to see all
options):

  - `-r, --recursive`: recurse into directories
  - `-v, --verbose`: increase verbosity
  - `-h, --human-readable`: human-readable format
  - `-z, --compress`: compress file data during the transfer
  - `-P, --partial --progress`: to keep partially transferred files
    which should make a subsequent transfer of the rest of the file much
    faster

For example:

``` bash
rsync -rPz ./home/myfiles user@remote.edu:./myproject
```

Will transfer files in “partial” mode from `./home/myfiles/` in the
local machine to the remote `./myproject` directory. Additionally,
compression will be used to reduce the size of data portions of the
transfer.

### SSH file transfer protocol (sftp)

The other method to transfer data between the local machine and a
cluster is SSH file transfer protocol or `sftp`. The great advantage of
this way is tab completion in both local and remote that makes finding
origins and destinations much easier. We can connect to a cluster
through `sftp` very similar to `ssh` by running `sftp username@server`.

We can also use most of the Bash commands when using `sftp` and at the
same time access to both cluster and local computer. Usually we can
apply the bash commands in the local system by adding `l` to the
beginning of the commands. For example:

``` bash
pwd # print working directory on the cluster
lpwd # print working directtory on the local computer
cd # chnage directory on the cluster
lcd # change directtory on the local computer
```

Use `put` to upload and `get` to download a file.

### Wget

To get data from web, also, we can use `wget` command to download files
on the cluster.

-----

## Software installation

When we login to a cluster, as a user we only have permission to change
user level files (home directory `cd ~` and higher). So, in this case,
we never be able to install/update software that are located in the root
directory (`cd /`). Note that we can find the location of software by
`module show <software-name>` command.

As a cluster user, we have several ways to build our own system and
install and update our required software:

1.  **Python**: If we only need several Python packages probably the
    easiest way is making a virtual environment by `venv` module in
    Python3. After that we will be able to use `pip` package manager to
    install packages.

2.  **Miniconda**: It let us install many software including Python, R
    and their packages. We can try `module load anaconda3` to load the
    module and then use `conda` to create a virtual environment and
    install software and packages. Note that if the cluster does not
    include `miniconda3`, then you may use the third option to install
    it first. Review [Virtual environments in Python](./python-env.html)
    to learn more.

3.  **Spack**: it gives more variety of software and packages to install
    (see
    [here](https://spack.readthedocs.io/en/latest/package_list.html#gurobi)).
    To use Spack, we need to install it on a local directory (which is
    `cd ~` and above) and then use `spack` to install and load packages.
    Note that this way might take more time to install Spack and
    required modules, so, first make sure the second option could not
    install your requirements. Review [Install software with
    Spack](./spack.html) to learn more.

4.  **Manually**: Still there are many software that are not available
    through Conda or Spack. We should follow the software instruction to
    install them. Make sure to review README or INSTALL file (if exist)
    and check configure options, `./configure --help`, in the
    installation directory. Since, we are not using root directory, make
    sure to use the right directory that all the dependencies already
    installed (i.e `./configure --prefix=${PWD}`). This method could be
    the hardest way, so, first make sure Conda and Spack could not help
    you. Note that software names might be slightly different in Conda
    or Spack, so have a look on all names that are close.

---

Copyright 2018-2023, [Ashkan Mirzaee](https://ashki23.github.io/index.html) | Content is available under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) | Sourcecode licensed under [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
