---
layout: post
title: Setting up an APT cache
subtitle: Speeding up upgrades
date: "2024-06-03T00:00:00Z"

tags: [tutorial, APT, LXC, LXD, ROS, ROS 2]
---

I've been advocating in several past posts for the use of [LXD] as a virtual development environment - see for instance ["Setting up ROS 2 Jazzy with LXD"][jazzy-post].
But when setting up such environment for ROS 2,
we find ourselves downloading hundreds of packages which may take some times.

There are several ways we can address this and we are going to explore one here that has the advantages of being rather simple and generalise beyond LXD and ROS 2.
That solution is to cache our packages in an apt proxy.

In this post we're going to make use of [LXD],
[cloud-init][cloud-init-doc] and adresse ROS 2 specifically but none of those are absolutely necessary and a similar setup can be replicated in a different scenario.

Let's get started.

## Creating a proxy

First thing first,
we shall set up a container that will act as the apt proxy and locally cache the debs.

So let's start by creating said container:

```bash
lxc launch ubuntu:24.04 apt-proxy
```

Once instantiated,
we can then shell into the container and update/upgrade it for good measure:

```bash
lxc shell apt-proxy
root@apt-proxy:~$ apt update && apt upgrade -y
```

Hereafter, the commands are run inside the 'apt-proxy' container.

In our proxy, we will use the [apt-cacher-ng][apt-cacher-ng-home] project.
To best describe apt-cacher-ng, let's quote it's documentation:

> [apt-cacher-ng is a] caching proxy. Specialized for package files from Linux distributors, primarily for Debian (and Debian based) distributions

And at this point, that's pretty much all we need to know as it is pretty much 'install & forget'. So let's do just that.

To install it enter:

```bash
apt install apt-cacher-ng
```

After that, make sure that the service is running:

```bash
$ systemctl status apt-cacher-ng
● apt-cacher-ng.service - Apt-Cacher NG software download proxy
     Loaded: loaded (/usr/lib/systemd/system/apt-cacher-ng.service; enabled; preset: enabled)
     Active: active (running)
...
```

And that's it, the caching mechanism it set up and listening by default on port 3142.

Before going any further,
wouldn't it be nice if this very container used the caching mechanism?
To do so, create the file:

```bash
vi /etc/apt/apt.conf.d/02proxy
```

with the following content:

```txt
Acquire::http { Proxy "http://127.0.0.1:3142"; };
```

With this, the apt-proxy LXD container will use the caching as well.

I've created a cloud-init config [available on GitHub][cloud-init-apt-proxy] to easily launch this proxy container with the following one-liner:

```bash
lxc launch ubuntu:24.04 apt-proxy --config=user.user-data="$(curl -L https://gist.githubusercontent.com/artivis/f1b201ae78fd182cc6c6dccd0abd0fa1/raw/apt-proxy.cloud-init.yaml)"
```

### Keeping an eye

Before moving on, let's point out that we can keep an eye on the logs of the cacher as they are produced.
This will be helpfull to make sure that everything goes as planed.
To do so, enter the following in a terminal:

```bash
tail -f /var/log/apt-cacher-ng/apt-cacher.log
```

## Creating a client

Typically, a cloud-init configuration for ROS 2 is something along the lines of the following:

```yaml
#cloud-config

apt:
  sources:
    ros2:
      source: "deb [arch=amd64] http://packages.ros.org/ros2/ubuntu {{ v1.distro_release }} main"
      keyid: C1CF 6E31 E6BA DE88 68B1 72B4 F42E D6FB AB17 C654

package_upgrade: true

packages:
  - build-essential
  - python3-colcon-common-extensions
  - ros-jazzy-ros-base
...
```

Installing ROS 2, whether a minimal install or a full desktop one,
will usually install a lot of packages.
That the whole reason we're setting up a cache in the first place!

So how can we point our new ROS 2 container to the proxy?
The simplest solution is to install the [auto-apt-proxy] package.
This project is pretty much a bash script that "autodetect common [local] APT proxy setups".

To install it, we add the following to our cloud-init config:

```diff
#cloud-config

+ bootcmd:
+   - [ cloud-init-per, once, apt-proxy-aptupdate, apt-get, update ]
+   - [ cloud-init-per, once, apt-proxy-aptinstall, apt-get, install, auto-apt-proxy ]
```

This will make sure that the auto-apt-proxy package is be installed sufficiently early in the cloud-init process that subsequent calls to apt (such as the one operated by the 'packages' keyword) will hit our proxy and thus our apt cache.
All of that happens rather automagically.

Note that all in all, we only install the `auto-apt-proxy` package.

Alright, let us make sure that everything work.
First we shall create a client container:

```bash
lxc launch ubuntu:24.04 jazzy-lxc --config=user.user-data="$(curl -L https://gist.githubusercontent.com/artivis/0357fe03a5ae459bee8c55823fbb0af8/raw/ros2.cloudinit.yaml)"
```

Note that I'm using in the command above a cloud-init configuration file that is [available on GitHub][cloud-init-jazzy].

While the container is instantiating,
let's have a look on our cacher logs:

```bash
tail -f /var/log/apt-cacher-ng/apt-cacher.log

1720023357|I|14844|10.203.148.193|archive.ubuntu.com/ubuntu/dists/noble/InRelease
1720023357|O|102|10.203.148.193|archive.ubuntu.com/ubuntu/dists/noble/InRelease
1720023357|I|14844|10.203.148.193|archive.ubuntu.com/ubuntu/dists/noble-updates/InRelease
1720023357|O|102|10.203.148.193|archive.ubuntu.com/ubuntu/dists/noble-updates/InRelease
1720023360|I|5159|10.203.148.193|ros2/dists/noble/InRelease
1720023360|O|4933|10.203.148.193|ros2/dists/noble/InRelease
1720023360|I|14844|10.203.148.193|security.ubuntu.com/ubuntu/dists/noble-security/InRelease
1720023360|O|102|10.203.148.193|security.ubuntu.com/ubuntu/dists/noble-security/InRelease
1720023360|I|14844|10.203.148.193|archive.ubuntu.com/ubuntu/dists/noble-backports/InRelease
1720023360|O|102|10.203.148.193|archive.ubuntu.com/ubuntu/dists/noble-backports/InRelease
1720023360|I|868602|10.203.148.193|ros2/dists/noble/main/binary-amd64/Packages.gz
1720023360|O|868326|10.203.148.193|ros2/dists/noble/main/binary-amd64/Packages.gz
1720023365|I|69304|10.203.148.193|ros2/pool/main/p/python3-colcon-core/python3-colcon-core_0.17.0-1_all.deb
1720023365|O|69018|10.203.148.193|ros2/pool/main/p/python3-colcon-core/python3-colcon-core_0.17.0-1_all.deb
1720023365|I|43798|10.203.148.193|ros2/pool/main/p/python3-catkin-pkg-modules/python3-catkin-pkg-modules_1.0.0-1_all.deb
1720023365|O|43527|10.203.148.193|ros2/pool/main/p/python3-catkin-pkg-modules/python3-catkin-pkg-modules_1.0.0-1_all.deb
1720023365|I|6844|10.203.148.193|ros2/pool/main/p/python3-colcon-alias/python3-colcon-alias_0.1.1-1_all.deb
1720023365|O|6562|10.203.148.193|ros2/pool/main/p/python3-colcon-alias/python3-colcon-alias_0.1.1-1_all.deb
1720023365|I|328416|10.203.148.193|archive.ubuntu.com/ubuntu/pool/main/n/node-jquery/libjs-jquery_3.6.1+dfsg+~3.5.14-1_all.deb
1720023365|O|328029|10.203.148.193|archive.ubuntu.com/ubuntu/pool/main/n/node-jquery/libjs-jquery_3.6.1+dfsg+~3.5.14-1_all.deb
...
```

Oh wow. There is a lot that got printed.
Upon closer inspection,
we can essentially notice that the first 12 lines correspond to an (apt) update,
while the following ones are packages being downloaded.
It looks like it works!

Another way to verify that is to shell inside the client container:

```bash
lxc shell jazzy-lxc
```

and call apt with some debug logs as follows:

```bash
$ apt update -o Debug::Acquire::http=true
Using auto proxy detect command: /usr/bin/auto-apt-proxy
auto detect command returned: 'http://10.203.148.160:3142'
Using auto proxy detect command: /usr/bin/auto-apt-proxy
auto detect command returned: 'http://10.203.148.160:3142'
Using auto proxy detect command: /usr/bin/auto-apt-proxy
auto detect command returned: 'http://10.203.148.160:3142'
0% [Working]GET http://security.ubuntu.com/ubuntu/dists/noble-security/InRelease HTTP/1.1
Host: security.ubuntu.com
Cache-Control: max-age=0
Accept: text/*
If-Modified-Since: Wed, 03 Jul 2024 14:01:45 GMT
User-Agent: Debian APT-HTTP/1.3 (2.7.14) non-interactive


GET http://packages.ros.org/ros2/ubuntu/dists/noble/InRelease HTTP/1.1
Host: packages.ros.org
Cache-Control: max-age=0
Accept: text/*
If-Modified-Since: Tue, 25 Jun 2024 14:39:05 GMT
User-Agent: Debian APT-HTTP/1.3 (2.7.14) non-interactive
...
```

where '10.203.148.160' should be the IP address of the `apt-proxy` container.

As before, we should see some new logs printed by the cacher,
logs that correspond to the update call.

At last, for extra good measure,
we can verify that our ROS 2 packages are indeed cached on our proxy server.
Do to so, we can enter the following command in the `apt-proxy` container:

```bash
$ ls /var/cache/apt-cacher-ng/packages.ros.org/pool/main/*/
/var/cache/apt-cacher-ng/ros2/pool/main/p/:
python3-catkin-pkg-modules  python3-colcon-cmake              python3-colcon-installed-package-information  python3-colcon-package-information  python3-colcon-recursive-crawl  python3-rosdistro-modules
python3-colcon-alias        python3-colcon-common-extensions  python3-colcon-mixin                          python3-colcon-package-selection    python3-colcon-ros              python3-rospkg-modules
python3-colcon-bash         python3-colcon-core               python3-colcon-notification                   python3-colcon-parallel-executor    python3-colcon-zsh              python3-vcstool
python3-colcon-cd           python3-colcon-defaults           python3-colcon-output                         python3-colcon-powershell           python3-rosdep
python3-colcon-clean        python3-colcon-devtools           python3-colcon-override-check                 python3-colcon-python-setup-py      python3-rosdep-modules

/var/cache/apt-cacher-ng/ros2/pool/main/r/:
ros-jazzy-action-msgs                             ros-jazzy-ament-lint-common           ros-jazzy-pybind11-vendor            ros-jazzy-ros2run                               ros-jazzy-rpyutils
ros-jazzy-actionlib-msgs                          ros-jazzy-ament-package               ros-jazzy-python-cmake-module        ros-jazzy-ros2service                           ros-jazzy-sensor-msgs
ros-jazzy-ament-cmake                             ros-jazzy-ament-pep257                ros-jazzy-rcl                        ros-jazzy-ros2topic                             ros-jazzy-sensor-msgs-py
ros-jazzy-ament-cmake-auto                        ros-jazzy-ament-uncrustify            ros-jazzy-rcl-action                 ros-jazzy-rosbag2                               ros-jazzy-service-msgs
ros-jazzy-ament-cmake-copyright                   ros-jazzy-ament-xmllint               ros-jazzy-rcl-interfaces             ros-jazzy-rosbag2-compression                   ros-jazzy-shape-msgs
ros-jazzy-ament-cmake-core                        ros-jazzy-builtin-interfaces          ros-jazzy-rcl-lifecycle              ros-jazzy-rosbag2-compression-zstd              ros-jazzy-shared-queues-vendor
...
```

Yep, there are plenty packages there.

## Some metrics

As a very scientific benchmark,
we are going to spawn the very same container,
with the same ROS 2 ready cloud-init configuration we used previously,
with and without the `apt-proxy` container running.

We recall the command to launch the container:

```bash
lxc launch ubuntu:24.04 jazzy-lxc --config=user.user-data="$(curl -L https://gist.githubusercontent.com/artivis/0357fe03a5ae459bee8c55823fbb0af8/raw/ros2.cloudinit.yaml)"
```

In both cases, we'll wait for cloud-init to finish:

```bash
lxc exec jazzy-lxc -- cloud-init status --wait
```

And as a measure, we will rely on cloud-init with:

```bash
lxc exec jazzy-lxc -- cloud-init analyze show
```

And the results are:

|   exec time (s)   | apt_configure | package_update_upgrade_install | total |
|------------|:-----:|:------:|:------:|
| w/o proxy  | 18.44 | 179.95 | 240.47 |
| with proxy | 03.57 |  39.59 |  90.66 |

The results are clear,
we observe a ~2.5x speed increase in apt operations.

Note that for good measure I ran this test a couple times.
The results between runs varies a bit of course but not significantly given the difference.

<!--
## On using mirrors

### Configuring the proxy for ROS 2 mirrors

We will create a file listing ROS 2 repository mirrors.

```bash
vi /etc/apt-cacher-ng/ros2_mirror
```

Then we edit the `apt-cacher-ng` configuration file:

```bash
vi /etc/apt-cacher-ng/acng.conf
```

and append the following:

```diff
+ file:ros2_mirror /packages.ros.org ; http://packages.ros.org/ros2/ubuntu
```
-->

## Conclusion

We have seen in this post how to set up an apt proxy to locally cache debs allowing us to speed up the instantiation of ROS 2 ready LXD containers.
And as we've seen, it is fairly simple. Gaining a ~2.5x speed increase in apt operations at the cost of 'fire & forget'ing a server container and installing a single package in our clients is a pretty good deal.

Remember that while we examplified the setup here using LXD and a ROS 2 dev environment,
this can be replicated in other scenario as well.

At last, the apt-cacher-ng comes loaded with features and options that you can define in the file `/etc/apt-cacher-ng/acng.conf`.
I leave it to you to explore that and tweak the cacher to your needs.

[//]: # (URLs)

[LXD]: https://canonical.com/lxd
[cloud-init-doc]: https://cloudinit.readthedocs.io/en/latest/index.html

[apt-cacher-ng-home]: https://www.unix-ag.uni-kl.de/~bloch/acng/
[auto-apt-proxy]: https://github.com/terceiro/auto-apt-proxy

[jazzy-post]: /post/2024/ros2-jazzy
[cloud-init-jazzy]: https://gist.github.com/artivis/0357fe03a5ae459bee8c55823fbb0af8
[cloud-init-apt-proxy]: https://gist.github.com/artivis/f1b201ae78fd182cc6c6dccd0abd0fa1
