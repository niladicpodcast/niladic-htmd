---
title: Setup a local SFTP server for development
author: Cody
published: 2018-01-10T00:00:00+00:00
_hash: 6b0f1017a5bd4eb3d9356de2f794d21c7173f66eed065002ec1340f1431e67f7
updated: 2026-04-07T09:03:47.412804+00:00
...

OpenSSH comes with an SFTP server. First install OpenSSH Server.

```shell
sudo apt install openssh-server
```

Create `sftponly` group.

```shell
sudo groupadd sftponly
```

Create a directory for SFTP files. The `ChrootDirectory` set in `sshd_config` must be owned by `root` and only writable by `root`.


```shell
sudo mkdir /sftp/
sudo chown root:sftponly /sftp/
sudo chmod 2755 /sftp/
# The leading 2 is for g+s
```

If the `ChrootDirectory` is not owned by `root` you will get the following error:
```
packet_write_wait: Connection to 127.0.0.1 port 22: Broken pipe
Couldn't read packet: Connection reset by peer
```

Configure SFTP server

- only available from localhost
- only available to users in the `sftponly` group
- `/sftp/` is the directory on disk

Add the following to the bottom of `/etc/ssh/sshd_config`

```
Match Group sftponly
    ChrootDirectory /sftp/
    ForceCommand internal-sftp
    AllowTcpForwarding no
```

```shell
sudo service ssh restart
```

You can create directories inside and copy files into them and they will be owned by the `sftponly` group.

```shell
sudo mkdir /sftp/files/
# Allow sftponly users to upload files
sudo chmod 2775 /sftp/files/
sudo cp file /sftp/files/
```

You can create a new user in the `sftponly` group and use that user and password to login to the SFTP server.

```shell
sudo useradd -M -G sftponly $newUser
sudo passwd $newUser
# set password
sudo -u $newUser sftp localhost
```


To add an existing user to the group:

```shell
sudo usermod -a -G sftponly $USER
```

In order for your user to actually be in the `sftponly` group you need to log in after the usermod command.
The safest thing to do is to compare your current groups with the new groups while you are still in the current environment and can still fix it.
I forgot the `-a` in the `usermod` command and had to use GRUB to get a root terminal and add my user back to the correct groups.

```shell
groups
id
```

Compare with the groups after login:

```shell
su - $USER
groups
id
```

The only difference should be the `sftponly` group.
