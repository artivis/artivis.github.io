---
layout: post
title: How to isolate an LXC
subtitle: On providing a remote sandbox
date: "2024-04-18T00:00:00Z"

tags: [tutorial, LXC, LXD]
---

In this post, we will see how to launch a Linux container that is isolated,
not only from other containers,
but also from the local network.
The container will however have access to internet.

Such sandboxed environment can come handy for many use cases.
For instance, I found myself in need to provide access to a development environment
running on a beefy machine of mine to a remote colleague.

While there are many ways to do this,
we will details here a setup that is fairly simple, straightforward and fully managed by LXD.
Most importantly, we won't have to deal with iptables.
However, not that it may not be suitable for complex setup,
nor does it scale well.

To follow along, we'll assume that you have [LXD installed][lxd-install],
and some basic experience with it.
Let us dive in.

## Create a dedicated network

LXC comes with powerful features when it comes to networking.
They may even be a little overwhelming to networking beginners such as myself.
Hence this post for futur reference.

First thing first, let us create a container using the default network,

```bash
lxc launch ubuntu:22.04 c0
```

Let us check that the container is up,

```bash
$ lxc list
+------+---------+----------------------+----------------------------------------------+-----------+-----------+
| NAME |  STATE  |         IPV4         |                     IPV6                     |   TYPE    | SNAPSHOTS |
+------+---------+----------------------+----------------------------------------------+-----------+-----------+
| c0   | RUNNING | 10.69.141.197 (eth0) | fd42:3bb:147e:3c99:216:3eff:fe4a:bd59 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+----------------------------------------------+-----------+-----------+
```

Looking at the existing network,

```bash
$ lxc network list
+--------+----------+---------+----------------+--------------------------+-------------+---------+---------+
|  NAME  |   TYPE   | MANAGED |      IPV4      |           IPV6           | DESCRIPTION | USED BY |  STATE  |
+--------+----------+---------+----------------+--------------------------+-------------+---------+---------+
| ens3   | physical | NO      |                |                          |             | 0       |         |
+--------+----------+---------+----------------+--------------------------+-------------+---------+---------+
| lxdbr0 | bridge   | YES     | 10.69.141.1/24 | fd42:3bb:147e:3c99::1/64 |             | 2       | CREATED |
+--------+----------+---------+----------------+--------------------------+-------------+---------+---------+
```

we can see that the container ip address is in the ip range of the `lxdbr0` network.

Alright, we'll keep this container aside for now and we will now create an LXD network dedicated to isolation.

To do so, issue the following,

```bash
lxc network create lxdbriso
```

This will create a new bridge called `lxdbriso`.

```bash
$ lxc network list
+----------+----------+---------+----------------+---------------------------+-------------+---------+---------+
|   NAME   |   TYPE   | MANAGED |      IPV4      |           IPV6            | DESCRIPTION | USED BY |  STATE  |
+----------+----------+---------+----------------+---------------------------+-------------+---------+---------+
| ens3     | physical | NO      |                |                           |             | 0       |         |
+----------+----------+---------+----------------+---------------------------+-------------+---------+---------+
| lxdbr0   | bridge   | YES     | 10.69.141.1/24 | fd42:3bb:147e:3c99::1/64  |             | 2       | CREATED |
+----------+----------+---------+----------------+---------------------------+-------------+---------+---------+
| lxdbriso | bridge   | YES     | 10.75.204.1/24 | fd42:4bb8:a4e4:9d16::1/64 |             | 0       | CREATED |
+----------+----------+---------+----------------+---------------------------+-------------+---------+---------+
```

We can note that `lxdbriso` has a different range than `lxdbr0`.
It is indeed a different network.

## Setting up some containers

At this point we can already launch a couple containers using the new network.
This will help us make sure that our setup progresses properly.

```bash
lxc launch ubuntu:22.04 c1 -n lxdbriso
lxc launch ubuntu:22.04 c2 -n lxdbriso
```

After launching the 2 containers we can list all containers again,

```bash
$ lxc list
+------+---------+----------------------+-----------------------------------------------+-----------+-----------+
| NAME |  STATE  |         IPV4         |                     IPV6                      |   TYPE    | SNAPSHOTS |
+------+---------+----------------------+-----------------------------------------------+-----------+-----------+
| c1   | RUNNING | 10.75.204.192 (eth0) | fd42:4bb8:a4e4:9d16:216:3eff:feac:1eea (eth0) | CONTAINER | 0         |
+------+---------+----------------------+-----------------------------------------------+-----------+-----------+
| c2   | RUNNING | 10.75.204.31 (eth0)  | fd42:4bb8:a4e4:9d16:216:3eff:fe40:5823 (eth0) | CONTAINER | 0         |
+------+---------+----------------------+-----------------------------------------------+-----------+-----------+
| c0   | RUNNING | 10.69.141.197 (eth0) | fd42:3bb:147e:3c99:216:3eff:fe4a:bd59 (eth0)  | CONTAINER | 0         |
+------+---------+----------------------+-----------------------------------------------+-----------+-----------+
```

and verify that `c1` & `c2` are on the `lxdbriso` network.

At this point, there is no isolation. You should thus be able to ping in any direction,

```bash
$ lxc exec c1 -- ping c2 -4 -c1
PING  (10.75.204.31) 56(84) bytes of data.
64 bytes from c2.lxd (10.75.204.31): icmp_seq=1 ttl=64 time=0.040 ms
...
$ lxc exec c1 -- ping 10.69.141.197 -4 -c1
PING 10.69.141.197 (10.69.141.197) 56(84) bytes of data.
64 bytes from 10.69.141.197: icmp_seq=1 ttl=63 time=0.075 ms
```

## Intra-net isolation

To isolate the two containers on the isolated network from each other,
we can use the [`port-isolation`][port-isolation] feature:

```bash
lxc config device set c1 eth0 security.port_isolation=true
lxc config device set c2 eth0 security.port_isolation=true
```

We shouldn't be able to ping from one to the other anymore,

```bash
$ lxc exec c1 -- ping c2 -4 -c1
PING  (10.75.204.31) 56(84) bytes of data.
From c1.lxd (10.75.204.192) icmp_seq=1 Destination Host Unreachable
...
```

This worked well.

> It is important to note that while very handy,
  the `port_isolation` feature solely
  "*prevent the NIC from communicating with other NICs in the network that have port isolation enabled*".
  Mind that a container may have several NICs,
  and that the feature would have to be enable for each new container since it works
  at a NIC level and not at a network level.

The intra-network isolation is in place.
Let's now look at inter-network isolation.

First, for good measure, let make sure we can still ping the container on the default network,

```bash
$ lxc exec c1 -- ping 10.69.141.197 -4 -c1
PING 10.69.141.197 (10.69.141.197) 56(84) bytes of data.
64 bytes from 10.69.141.197: icmp_seq=1 ttl=63 time=0.076 ms
...
```

Yup, still there.

## Inter-net isolation

To tackle the inter-network isolation we will use the [Access Control List (ACL)][acls] feature.

First we will set the default rule for all egress/ingress traffic,

```bash
lxc network set lxdbriso \
  security.acls.default.egress.action=allow \
  security.acls.default.ingress.action=allow
```

This essentially allow all traffic to/from our isolated network.
I know what you thinking, but let us keep going.

Next, we will create an ACL set of rules to actually isolate the network.
Create the ACL with,

```bash
lxc network acl create external-only
```

We can now create the first rule,

```bash
lxc network acl rule add external-only egress \
  destination=10.69.141.0/24 \
  action=reject
```

This rule specifies that all traffic at destination of `lxdbr0` should be blocked.
The second rule is very similar and will block all traffic going to our local network.

```bash
lxc network acl rule add external-only egress \
  destination=10.0.0.0/24 \
  action=reject
```

Note that you should replace `10.0.0.0` with your actual local network.
And of course more rules can be added if you need it.

With the ACL defined, we shall now apply it to our network,

```bash
lxc network set lxdbriso security.acls=external-only
```

Alright, let us try again to ping the container on the default network,

```bash
$ lxc exec c1 -- ping 10.69.141.197 -4 -c1
PING 10.69.141.197 (10.69.141.197) 56(84) bytes of data.
From 10.75.204.1 icmp_seq=1 Destination Port Unreachable
...
```

And the traffic is indeed blocked.
What about the local network,

```bash
$ lxc exec c1 -- ping 10.0.0.145 -4 -c1
PING 10.0.0.145 (10.69.141.197) 56(84) bytes of data.
From 10.75.204.1 icmp_seq=1 Destination Port Unreachable
...
```

Blocked as well. Note that here `10.0.0.145` is the ip of my host on the local network.
Replace it with you own.

At last, let's check that that we can still ping the world,

```bash
$ lxc exec c1 -- ping www.wikipedia.com -4 -c1
PING ncredir-lb.wikimedia.org (185.15.58.226) 56(84) bytes of data.
64 bytes from ncredir-lb.drmrs.wikimedia.org (185.15.58.226): icmp_seq=1 ttl=53 time=75.8 ms
...
```

Looks like we're all set.

## Isolate an existing container

This is all nice and shiny but what if you have an existing container on the default
network that you'd like to isolate?
Well worry not we can do that.

### Attach the isolated network

First, we have have to attach the isolated network to the container we would like to isolate,

```bash
lxc network attach lxdbriso c0 eth1
```

Let's check that the nic is properly attached,

```bash
$ lxc exec foo -- ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
17: eth0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:4a:bd:59 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.69.141.197/24 metric 100 brd 10.69.141.255 scope global dynamic eth0
       valid_lft 3409sec preferred_lft 3409sec
    inet6 fd42:3bb:147e:3c99:216:3eff:fe4a:bd59/64 scope global mngtmpaddr noprefixroute
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe4a:bd59/64 scope link
       valid_lft forever preferred_lft forever
19: eth1@if20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:57:5b:25 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fd42:4bb8:a4e4:9d16:216:3eff:fe57:5b25/64 scope global dynamic mngtmpaddr
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe57:5b25/64 scope link
       valid_lft forever preferred_lft foreve
```

Indeed it appears but note that it doesn't have an ipv4 address yet.

### Detach the initial network

We should now detach the initial default network.
The logic would press us to use something like:

```bash
lxc network detach lxdbr0 c0 eth0
```

However we're greeted with the following error message: `Error: The specified device doesn't exist`.
Nice.
What happens here is that the `eth0` device is inherited from the default lxd profile.
For some reasons those are mystical beasts that requires a specific handling.
That handling is as follows,

```bash
lxc config device add c0 eth0 none
```

You might be tempted by a 'what the' and I sympathise.
The explanation is that to remove a NIC inherited from a profile,
you actually have to mask it with an empty one.
That sounds like something that should be fixed right.
Anyhow, let us continue.

Our container is now attached to the isolated network.
But it still doesn't have an ipv4 address.
To fix that will have to edit its network configuration.

Let us shell into the container,

```bash
$ lxc exec c0 -- sudo --login --user ubuntu
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@c0:~$
```

and edit its netplan configuration file,

```bash
vi /etc/netplan/50-cloud-init.yaml
```

where we will change the following,

```diff
network:
    version: 2
    ethernets:
-        eth0:
+        eth1:
            dhcp4: true
```

Save, exit and apply the new configuration,

```bash
netplan apply
```

In retrospect, we probably could have attached the isolated network directly as `eth0` with,

```bash
lxc network attach lxdbriso c0 eth0
```

but where is the fun in that?

### Is it isolated yet?

At this point, we have to remember that the intra-net traffic is blocked on a per-NIC basis:

```bash
$ lxc exec c0 -- ping c2 -4 -c1
PING  (10.75.204.31) 56(84) bytes of data.
64 bytes from c2.lxd (10.75.204.31): icmp_seq=1 ttl=64 time=0.047 ms
...
```

We shall thus set enable `port_isolation` for `c0`'s new NIC,

```bash
lxc config device set c0 eth1 security.port_isolation=true
```

Let's check that it works,

```bash
$ lxc exec c0 -- ping c2 -4 -c1
PING c2.lxd (10.75.204.31) 56(84) bytes of data.
From c0.lxd (10.75.204.226) icmp_seq=1 Destination Host Unreachable
...
```

Let's verify the inter-net traffic for good measure,

```bash
$ lxc exec c0 -- ping 10.0.0.145 -4 -c1
PING 10.0.0.145 (10.0.0.145) 56(84) bytes of data.
From 10.75.204.1 icmp_seq=1 Destination Port Unreachable
...
```

We can't ping the local network. What about the greater internet,

```bash
$ lxc exec foo -- ping wikipedia.com -4 -c1
PING wikipedia.com (185.15.58.226) 56(84) bytes of data.
64 bytes from ncredir-lb.drmrs.wikimedia.org (185.15.58.226): icmp_seq=1 ttl=53 time=30.0 ms
...
```

Yep that works.

## A note on Docker

If you are not using Docker alongside LXD on the same host,
you may skip this note altogether.
If you are, well you probably already now that there is a network-related issue in having both installed.
Chance is, you've already seen the [related documentation][docker-note].

This only a reminder that if you went with the iptables solution,
you should also allow the traffic for `lxdbriso`.
Don't ask me why I feel the need to remind you about that.

## Conclusion

We've seen in this post how we can isolate a Linux Container so that it can only reach out to internet.
As stated at the beginning of this post,
there are plenty other ways to do that, from messing around with iptables
to dedicated vlans on your fancy ethernet switch.
I went down this road as it fairly simple and fully managed through LXD.

In introduction to this post I also told you what was my initial motivation for this setup.
To provide access to a playground to a remote colleague.
But I didn't touch on that aspect did I?
Well that wasn't quite the point here but I'll still leave you with a clue to explore:

```bash
snap install netbird
```

> Disclaimer: this post was only made possible thanks to the LXD's online documentation and forum. Most notably, it is an expanded version of [a comment][comment] from [Thomas Parrott][tomp]. Credits where they're due.

[//]: # (URLs)

[lxd-install]: https://documentation.ubuntu.com/lxd/to/latest/installing/
[port-isolation]: https://documentation.ubuntu.com/lxd/en/latest/reference/devices_nic/#device-nic-bridged-device-conf:security.port_isolation
[acls]: https://documentation.ubuntu.com/lxd/en/latest/howto/network_acls/
[docker-note]: https://documentation.ubuntu.com/lxd/en/latest/howto/network_bridge_firewalld/#prevent-connectivity-issues-with-lxd-and-docker
[tomp]: https://discuss.linuxcontainers.org/u/tomp/summary
[comment]: https://discuss.linuxcontainers.org/t/prevent-cross-talk/14745/8
