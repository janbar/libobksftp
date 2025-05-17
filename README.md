# LibobkSFTP

## What is it ?

It is a SBT driver for ORACLE® RMAN allowing to backup and restore database
from any SSH server. The transfer protocol is highly secure using SFTP over
SSH2. Thus, you can backup the database anywhere, without the need for local
storage, or share, or expensive third-party software.

## How it works ?

The driver implements the SBT API V2 and acts as a SFTP client to list or
transfer the backup set. It supports multiple channels, and the transfer rate
is not limited.
Authentication with the remote server uses only SSH key exchange (KEX). You can
use the default key pair (id_rsa) of the oracle user, or a dedicated pair. The
remote server only needs to know the public key and it is not allowed to
connect to the database server.

## What dependency ?

The driver is statically linked with LIBSSH2, and dependencies are limited to
OpenSSL which is provided with the OS distribution. You don't need any agent or
third-part software. All requirements are in your database server box.

## Warning

All interfaces are based on reverse engineering of RMAN SBT. Since the SBT API
isn't public, you have no way to obtain it by free, so you use this at your own
risk.

## Test it

Remote user must append its file `~/.ssh/authorized_keys` with the public key
of the local oracle user i.e `~/.ssh/id_rsa.pub`.
Using the default `$HOME/.ssh/` keys requires that the `HOME` environment variable be set correctly. Otherwise you must specify the paths with `OB_PUBLIC_KEY` and `OB_SECRET_KEY`.

```
RUN {
ALLOCATE CHANNEL S1 DEVICE TYPE sbt
PARMS='SBT_LIBRARY=/usr/local/lib/libobksftp.so,ENV=(
OB_SERVER=<remote host>,
OB_USER=<remote user>)';

BACKUP CURRENT CONTROLFILE FORMAT 'rman/%d/%U';
RESTORE CONTROLFILE VALIDATE;
}
```

## Configure the channel sbt

The sample below, set an absolute path for prefix the format.

```
CONFIGURE CHANNEL DEVICE TYPE sbt FORMAT '%d/%U'
PARMS='SBT_LIBRARY=/usr/local/lib/libobksftp.so,ENV=(
OB_SERVER=<remote host>,OB_USER=<remote user>,OB_PATH=/backup)';

CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE sbt TO '%d/%F';

CONFIGURE DEFAULT DEVICE TYPE TO 'SBT_TAPE';
```

## ENV settings

```
OB_SERVER     : REQUIRED, remote server name or IP
OB_USER       : REQUIRED, remote user owns of the backup pieces
OB_PATH       : optional, prefix path of the storage    (default: remote user home)
OB_PUBLIC_KEY : optional, public key file path to use   (default: ~/.ssh/id_rsa.pub)
OB_SECRET_KEY : optional, private key file path to use  (default: ~/.ssh/id_rsa)
OB_PORT       : optional, remote server port            (default: 22)
OB_LOGFILE    : optional, file path for debug logging   (default: none)
```

Note: The OB_LOGFILE parameter is used to trace internal errors (OBK and SSH2) for debugging. **It should only be set for testing with the SBTTEST tool**.

## Annexes

Obviously, you can easily list the sbt backup pieces on a remote server, using the ssh command. that might help to populate the catalog by generating a script as follows.

```
for bkp in $(ssh ${OB_USER}@${OB_SERVER} "ls ${OB_PATH}"); do
  echo "CATALOG DEVICE TYPE 'SBT_TAPE' BACKUPPIECE '$bkp';"
done
```

