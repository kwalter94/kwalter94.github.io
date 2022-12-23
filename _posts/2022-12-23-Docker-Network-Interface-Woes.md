---
title: Docker Network Interface Woes
category: [System administration]
tags: [docker, firewalld, linux, system administration]
date: 2022-12-23 11:30
---

After some system updates on my Opensuse Tumbleweed box, dockerd could not
start. Any attempts to start it would be responded to with the following
error:

    WARN[2022-12-23T13:17:02.715976321+02:00] could not create bridge network for id befcb314f3a719effeaf8ab7910123a5c20528603e20d437a42163583523b898 bridge name docker0 while booting up from persistent state: Failed to program NAT chain: ZONE_CONFLICT: 'docker0' already bound to 'internal'
    failed to start daemon: Error initializing network controller: Error creating default "bridge" network: Failed to program NAT chain: ZONE_CONFLICT: 'docker0' already bound to 'internal'

My first thought was to maybe delete the `docker0` network interface.
So, I went ahead and deleted it using:

```sh
sudo ip link delete docker0
```

Unfortunately, this didn't resolve the issue. A bit of Googling led me to
understand that I was looking at the wrong place. The problem was on the
firewall configuration. To verify this, I had to run the following:

```sh
$ firewall-cmd --get-active-zones
docker
  interfaces: br-7a20169cf60a
internal
  interfaces: docker0
public
  interfaces: enp0s20f0u1
```

From the output of the command above, the source of the error becomes clear.
The old docker daemon was using the `docker0` interface under the `internal`
zone, whilst the new docker daemon is trying to create a new `docker0` interface
under the `docker` zone. I tried a couple of solutions before I landed on the
[correct one](https://gist.github.com/reytech-dev/1cbbb158df374018be454537de32a428).
The following is what I tried and didn't work:

```sh
$ sudo firewall-cmd --permanent --zone=internal --remove-interface=docker0
$ sudo firewall-cmd --reload
# The above didn't work

$ sudo firewall-cmd --permanent --zone=docker --change-interface=docker0
$ sudo firewall-cmd --reload
# This too, didn't work
```

Close to giving up, I run into
[this gist](https://gist.github.com/reytech-dev/1cbbb158df374018be454537de32a428).
I noted that the author was pretty much doing the same as I was doing
with one exception, they weren't simply reloading the firewall rules. They
were doing a complete restart. So, I tried that and it worked. If you run
into this issue again Mr Ntumbuka, please try this first and if it doesn't
work, go ahead and repeat everything above:

```sh
$ sudo firewall-cmd --permanent --zone=docker --change-interface=docker0
$ sudo systemctl restart firewalld
```
