---
title: Setting up ZNC IRC bouncer on ubuntu 16.04
date: '2018-02-03 0:00:00'
layout: post
---

HOME
ABOUT
ARCHIVE
LINKEDIN
GITHUB
How to install ZNC with the playback module on Ubuntu 16.04
Posted by Andrew Stambrosky on November 19, 2016
IRC is great but I wanted to stop spamming my friends with join/parts. That’s when I learned about ZNC, which allows you to remain connected to IRC and not miss a thing. It is great but it can be tricky to set up.

I have multiple devices and I really wanted to be able to switch between them pretty seemlessly. A problem with ZNC by default is that either the buffer will be replayed even when you already have it or will be cleared so that you can’t switch between devices without losing history. That is why some IRC clients are supporting the Playback module which is here http://wiki.znc.in/Playback This allows the client to recieve only what it hasn’t recieved yet. Meaning no more duplicated buffers and no more history locked to a single device.

When I was trying to set this up I couldn’t find an all-in-one guide which explained everything so it took me a few days to figure it all out.

So first let’s get our system up to date and install the prerequisites.

ZNC is in the main Ubuntu repos so we can just install it and znc-dev like this

root@ubuntu-znc:~# apt-get update
root@ubuntu-znc:~# apt-get upgrade
root@ubuntu-znc:~# apt-get install znc znc-dev
We will need znc-dev for compiling and installing the playback module as well as any other modules not included by default.

During the next bit we will be following the steps to set this up as a system service. Ubuntu 16.04 uses systemd so we will follow those steps from the following wiki page.

http://wiki.znc.in/Running_ZNC_as_a_system_daemon

We will also be setting up ZNC, I set it up with some generic settings, they are all mostly easily changeable after you deploy as well.

root@ubuntu-znc:~# sudo useradd --create-home -d /var/lib/znc --system --shell /sbin/nologin --comment "Account to run ZNC daemon" --user-group znc
root@ubuntu-znc:~# sudo -u znc /usr/bin/znc --datadir=/var/lib/znc --makeconf
[ .. ] Checking for list of available modules...
[ >> ] ok
[ ** ]
[ ** ] -- Global settings --
[ ** ]
[ ?? ] Listen on port (1025 to 65534): 6697
[ ?? ] Listen using SSL (yes/no) [no]: yes
[ ?? ] Listen using both IPv4 and IPv6 (yes/no) [yes]: yes


[Unit]
[ .. ] Verifying the listener...
[ >> ] ok
[ ** ] Unable to locate pem file: [/var/lib/znc/znc.pem], creating it
[ .. ] Writing Pem file [/var/lib/znc/znc.pem]...
[ >> ] ok
[ ** ] Enabled global modules [webadmin]
[ ** ]
[ ** ] -- Admin user settings --
[ ** ]
[ ?? ] Username (alphanumeric): admin
[ ?? ] Enter password:
[ ?? ] Confirm password:
[ ?? ] Nick [admin]:
[ ?? ] Alternate nick [admin_]:
[ ?? ] Ident [admin]:
[ ?? ] Real name [Got ZNC?]: admin
[ ?? ] Bind host (optional):
[ ** ] Enabled user modules [chansaver, controlpanel]
[ ** ]
[ ?? ] Set up a network? (yes/no) [yes]: yes
[ ** ]
[ ** ] -- Network settings --
[ ** ]
[ ?? ] Name [freenode]:
[ ?? ] Server host [chat.freenode.net]:
[ ?? ] Server uses SSL? (yes/no) [yes]:
[ ?? ] Server port (1 to 65535) [6697]:
[ ?? ] Server password (probably empty):
[ ?? ] Initial channels: #znc
[ ** ] Enabled network modules [simple_away]
[ ** ]
[ .. ] Writing config [/var/lib/znc/configs/znc.conf]...
[ >> ] ok
[ ** ]
[ ** ] To connect to this ZNC you need to connect to it as your IRC server
[ ** ] using the port that you supplied.  You have to supply your login info
[ ** ] as the IRC server password like this: user/network:pass.
[ ** ]
[ ** ] Try something like this in your IRC client...
[ ** ] /server <znc_server_ip> +6697 admin:<pass>
[ ** ]
[ ** ] To manage settings, users and networks, point your web browser to
[ ** ] https://<znc_server_ip>:6697/
[ ** ]
[ ?? ] Launch ZNC now? (yes/no) [yes]: no
root@ubuntu-znc:~# touch /etc/systemd/system/znc.service
root@ubuntu-znc:~# vim /etc/systemd/system/znc.service
We’ll put the following in the systemd unit file /etc/systemd/system/znc.service.

[Unit]
Description=ZNC, an advanced IRC bouncer
After=network-online.target

[Service]
ExecStart=/usr/bin/znc -f --datadir=/var/lib/znc
User=znc

[Install]
WantedBy=multi-user.target
Right now I’m going to start the service and set it to start on boot.

root@ubuntu-znc:~# systemctl start znc.service
root@ubuntu-znc:~# systemctl enable znc.service
Created symlink from /etc/systemd/system/multi-user.target.wants/znc.service to /etc/systemd/system/znc.service.
Now we can compile and install the playback module. Compiling is easy, the znc-dev package gave use the command znc-buildmod to use which compiles a module in a good format for installing to znc. Based on the systemd unit file we know the datadir is /var/lib/znc. Most other tutorials will tell you to install the module ine ~/.znc/modules because ~/.znc is the datadir when you don’t set it up as a service. We have though so we’ll need to create a directory /var/lib/znc/modules/.

root@ubuntu-znc:~# git clone https://github.com/jpnurmi/znc-playback.git
Cloning into 'znc-playback'...
remote: Counting objects: 216, done.
remote: Total 216 (delta 0), reused 0 (delta 0), pack-reused 216
Receiving objects: 100% (216/216), 44.03 KiB | 0 bytes/s, done.
Resolving deltas: 100% (85/85), done.
Checking connectivity... done.

root@ubuntu-znc:~# cd znc-playback/

root@ubuntu-znc:~/znc-playback# znc-buildmod playback.cpp
Building "playback.so" for ZNC 1.6.3... [ ok ]



root@ubuntu-znc:/var/lib/znc# mkdir /var/lib/znc/modules
root@ubuntu-znc:/var/lib/znc# cp /root/znc-playback/playback.so /var/lib/znc/modules/
root@ubuntu-znc:/var/lib/znc# systemctl restart znc.service

ZNC is now running and the playback module can be enabled from the web interface.

From here there are many other great resources to setting up and tuning ZNC just how you’d like it

This is another great blog post which helped me set up znc with my favorite client, Textual.

https://ertw.com/blog/2016/01/09/configuring-textual-5-with-znc-and-playback/

← PREVIOUS POST

Copyright © Andrew Stambrosky 2016
