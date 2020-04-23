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

The following steps have been performed on *your* Client e.g Windows, MacOS or Linux client. If necessary, adjust the commands, filenames or the host name according to your environment.

- Start a *Putty* session from command line to log into your jump host. Replace **NN** with the number of you host.

```cmd
putty -ssh opc@osec-jumpNN.trivadislabs.com -i keys/id_rsa_osec-jumpNN.ppk
```

- Start a *SSH* session from command line. Replace **NN** with the number of you host.

```bash
ssh opc@osec-jumpNN.trivadislabs.com -i keys/id_rsa_osec-jumpNN
```

- Use a *SSH* proxy to directly access the DB server. Replace **NN** with the number of you host.

```bash
ssh -i keys/id_rsa_osec-jumpNN -YA -o ProxyCommand="ssh -W %h:%p opc@osec-jumpNN.trivadislabs.com" oracle@10.0.1.6
```

- Use a *SSH* proxy to directly access the DB server. Replace **NN** with the number of you host.

```bash
ssh -i keys/id_rsa_osec-jumpNN -YA -o ProxyCommand="ssh -W %h:%p opc@osec-jumpNN.trivadislabs.com" oracle@10.0.1.5
```

- Setup port forwarding for MS Remote Desktop via *SSH* or *PUTTY*. Replace **NN** with the number of you host.

```bash
ssh -A -L 33890:10.0.1.4:3389 opc@osec-jumpNN.trivadislabs.com

putty.exe -ssh -A -i id_rsa_osec-jumpNN.ppk -L 33890:10.0.1.4:3389 opc@osec-jumpNN.trivadislabs.com
```

- Login to the AD server `10.0.1.4` or `ad.trivadislab.com` using MS remote desktop over port forwarding.

!["RDP Dialog MacOS"](images/rdb_connect.png)

</div>
