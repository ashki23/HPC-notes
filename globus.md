# Globus: unified access to our research data

[Globus](https://www.globus.org) is a platform that let us to get unified access to our research data, across all systems, using any existing identity. It is very simple to start using Globus. Review [here](https://docs.globus.org/how-to/globus-connect-personal-linux/) to learn how to install and configure Globus connect personal for Linux.

In general we need to configure Globus and create personal endpoints such that:

### In Globus:
Login at [app.globus.org](https://app.globus.org/).

### At the endpoint:
```bash
## Install globus
wget https://downloads.globus.org/globus-connect-personal/linux/stable/globusconnectpersonal-latest.tgz
tar -xzvf globusconnectpersonal-latest.tgz
cd globusconnectpersonal-3.1.5

## Setup
./globusconnectpersonal -setup --no-gui
# Login and copy the auth code
# Enter the auth code
# Select the Endpoint name

## Start
./globusconnectpersonal -start
```

Note that Globus is mounting home directory by default. You may change the default path by updating `~/.globusonline/lta/config-paths`. Add a line with the absolute path that you want for globus to this file – save and close. By default it is `~/,0,1` the additional values of 0 and 1 indicate sharing path (premium feature so I used 0) and r/w (1 for writable, 0 for read-only).

---
Copyright 2018-2019, [Ashkan Mirzaee](https://ashki23.github.io/index.html) | Content is available under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) | Sourcecode licensed under [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
