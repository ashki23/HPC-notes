# Globus: unified access to our research data

[Globus](https://www.globus.org) is a platform that let us to get unified access to our research data, across all systems, using any existing identity. It is very simple to start using Globus. Review [here](https://docs.globus.org/how-to/globus-connect-personal-linux/) to learn how to install and configure Globus connect personal for Linux.

In general we need to configure Globus and the endpoint such that:

### In globus:
1. Login at [globus.org](https://www.globus.org) - you can use your `XSEDE` account
2. Go to ENDPOINTS > + Create a personal endpoint > ... and copy the setup (GCP) key at the end

### In the endpoint:
```bash
## Instal globus
wget https://downloads.globus.org/globus-connect-personal/linux/stable/globusconnectpersonal-latest.tgz
tar -xzvf globusconnectpersonal-latest.tgz
cd globusconnectpersonal*

## Start 
./globusconnectpersonal -setup <paste the setup (GCP) key>
./globusconnectpersonal -start
```

---
Copyright 2018-2019, [Ashkan Mirzaee](https://ashki23.github.io/index.html) | Content is available under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) | Sourcecode licensed under [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
