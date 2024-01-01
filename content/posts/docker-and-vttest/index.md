---
title: "Docker and VT Test"
date: 2020-01-03
slug: "/docker-and-vttest"
description: TBD
tags:
  - Testing
  - Docker
---
Docker is such a powerful tool to use during development and testing. One use case I had recently was to use docker in testing
a [VT220](https://vt100.net/docs/vt220-rm/) emulator that I wrote. The test cases involved running the emulator against the
[vttest](https://invisible-island.net/vttest/vttest.html) application to validate the parsing of the escape sequences and data
from an ssh server and the rendering of a screen from that server. I created a centos container with the vttest application
installed and sshd, so that I could ssh into that container and run the vttest application during my tests. Vttest is available
through an rpm package, so my first step was to create a dockerfile to create the centos container, download the rpm package,
and install openssh and vttest. Since this container is only going to be used during development and testing, I just configured
it for login as root by username and password. The dockerfile looked like this.

```
FROM centos:8

RUN curl https://download-ib01.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm > epel-release-6-8.noarch.rpm

RUN rpm -Uvh epel-release*rpm

RUN yum -y install openssh openssh-clients openssh-server vttest 
RUN sed -i s/#PermitRootLogin.*/PermitRootLogin\ yes/ /etc/ssh/sshd_config \
   && echo "root:root" | chpasswd

EXPOSE 22
COPY entrypoint.sh /
RUN chmod +x entrypoint.sh
CMD ["/entrypoint.sh"]
```

My entrypoint.sh file just generated the ssh key for a sshd server and started the server.

```
#!/bin/bash
# generate host keys if not present
ssh-keygen -A
# do not detach (-D), log to stderr (-e), passthrough other arguments
/usr/sbin/sshd -D -e "$@"
```

Next was to build the image and start a container.

```
docker build -t vttest .
docker run -rm -d -p 8222:22 vttest
```

Then I was able to ssh into the container…

```
Last login: Thu Jan  9 16:37:04 on ttys005

The default interactive shell is now zsh.
To update your account to use zsh, please run `chsh -s /bin/zsh`.
For more details, please visit https://support.apple.com/kb/HT208050.
Brians-MacBook-Pro:~ bhnat$ ssh -p 8222 root@localhost
The authenticity of host '[localhost]:8222 ([::1]:8222)' can't be established.
ECDSA key fingerprint is SHA256:FurbJ7Zp/hRTDZjuxZD61tw3EEeJVeBIGTwVPkbJ7AI.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[localhost]:8222' (ECDSA) to the list of known hosts.
root@localhost's password:
"System is booting up. Unprivileged users are not permitted to log in yet. Please come back later. For technical details, see pam_nologin(8)."
[root@c4cd578aa1a2 ~]#
```

…and start the vttest application to verify everything worked properly.

```
        VT100 test program, version 2.7 (20140305)
        Line speed 38400bd
        Choose test type:

         0. Exit
         1. Test of cursor movements
         2. Test of screen features
         3. Test of character sets
         4. Test of double-sized characters
         5. Test of keyboard
         6. Test of terminal reports
         7. Test of VT52 mode
         8. Test of VT102 features (Insert/Delete Char/Line)
         9. Test of known bugs
         10. Test of reset and self-test
         11. Test non-VT100 (e.g., VT220, XTERM) terminals
         12. Modify test-parameters

         Enter choice number (0 - 12):
```

Now I had a test environment ready for the tests of my VT220 emulator. As the emulator is part of an application that runs many
concurrent ssh client sessions, the next step will be to set this up in a kubernetes cluster, similar to a
[kubernetes jump server](https://github.com/kubernetes-contrib/jumpserver), but with the vttest application installed.
