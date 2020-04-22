## Solution 1: Get known the Environment

The following steps are performed in this exercise:

- login via SSH client as user *opc* to the individual OCI jump host e.g. `osec-jumpNN.trivadislabs.com`.
- Login to the DB and OUD server.
- Setup SSH port forwarding for remote desktop (3389).
- Login to the AD server using MS remote desktop.
- Check the different directories.

<!-- Stuff between the <div class="notes"> will be rendered as pptx slide notes -->
<div class="notes">
</div>

<!-- Stuff between the <div class="no notes"> will not be rendered as pptx slide notes -->
<div class="no notes">

### Detailed Solution

Depending on the client OS and used SSH client the solution vary. The examples shown here is using raw SSH and putty commands.

- Login to your individual OCI compute instance via jump host `osec-jumpNN.trivadislabs.com`
- Login to the DB server `10.0.1.6` or `db.trivadislab.com` over jump host and/or `ssh` ProxyCommand.
- Login to the OUD server `10.0.1.5` or `oud.trivadislab.com` over jump host and/or `ssh` ProxyCommand.
- Setup SSH port forwarding for remote desktop (3389).
- Login to the AD server `10.0.1.4` or `ad.trivadislab.com` using MS remote desktop over port forwarding.
- Check environments on DB server and AD server. OUD is currently still rather empty.

The following steps have been performed on the *ol7docker00* host. If necessary, adjust the commands, filenames or the host name according to your environment.

- Start a Putty session from command line.

```bash
putty -ssh opc@ol7docker00.trivadislabs.com -i keys/id_rsa_ol7docker00.ppk
```

- Alternatively start a SSH session from command line

```bash
ssh opc@ol7docker00.trivadislabs.com -i id_rsa_ol7docker00
```

- Switch to user *oracle*

```bash
sudo su - oracle
```

- Run *docker images* to see which images are available

```bash
docker images
```

- Run *docker volumes* to see which volumes are available

```bash
docker volumes ls
```

- Check the different directories and aliases.

```bash
cd /u01/volumes
cdl
ex01
o-db-docker
```
</div>
