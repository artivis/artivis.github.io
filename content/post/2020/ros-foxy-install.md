---
layout: post
title: Get started with ROS 2 Foxy today with LXD
subtitle: Why wait?
date: "2020-06-06T00:00:00Z"

reading_time: true  # Show estimated reading time?
share: true  # Show social sharing links?
profile: false  # Show author profile?
comments: false  # Show comments?
tags: [tutorial, ROS 2, Foxy, LXC, LXD]
---

The 5th of June 2020 marks the release of ROS 2 Foxy Fitzroy,
a 3 years long-term support (LTS) release and
the first ROS 2 distribution to target [Ubuntu 20.04][20-04].

As summarized in the [ROS Discourse][discourse-post] post,
Foxy comes loaded with performance improvements, new features,
and maybe most importantly, with [tutorials][tuto]!
Get a full tour of the novelties by heading down to the
[Foxy release page][release-page].

Now, you may be very excited about ROS 2 Foxy but you,
just as I, haven't moved to Ubuntu 20.04 just yet.
But that will not stop us from getting our hands on
all the goodies this new release offers.

Indeed, in this post we will see how to install ROS 2 Foxy Fitzroy
in a [LXD][LXD] container so that we can develop against the latest ROS 2
release without the need to upgrade our computer just yet.

Hereafter we will assume that your are familiar with the command terminal
and that LXD is already installed on your machine.
If you are new to LXD or looking to improve your ROS development with it,
have a look to this post ['ROS Noetic development workflow in LXC'][LXD-post].

Alright let us get started.

---
# Content
*   [Spawning an Ubuntu 20.04 LXD container](#spawning-an-ubuntu-2004-lxd-container)
*   [Installing ROS 2 Foxy Fitzroy](#installing-ros-2-foxy-fitzroy)
*   [Quick test](#quick-test)
---

# Spawning an Ubuntu 20.04 LXD container

So the first thing we have to do is to create a LXD container based on an
Ubuntu 20.04 image. To do so we issue the following command,
```bash
lxc launch ubuntu:20.04 ros2-foxy
```
It creates and starts a container named 'ros2-foxy' based on an
Ubuntu 20.04 image. So far so good.

Now to start a shell in our fresh container we will type,
```bash
lxc exec ros2-foxy -- sudo --login --user ubuntu
```
We are now inside our container, logged as the non-root user 'ubuntu'.
Note that if the last command looks a bit unfriendly to you,
you can make it a 'lxc' alias
(learn more about it in the [aforementioned LXD post][LXD-post-alias]).

Now that our container is up and running, we shall install Foxy.

# Installing ROS 2 Foxy Fitzroy

Inside our container, we will first add the ROS packages
repository to our sources. Starting with the key,

```bash
$ sudo apt-key adv --fetch-keys https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc
```
then the repository,
```bash
$ sudo apt-add-repository http://packages.ros.org/ros2/ubuntu
```

In case of trouble, you can also refer to the
[official installation guide][foxy-install].

We are all set to install Foxy!

For the installation, we can choose either of two options;
we can choose to install only the base components,
e.g. the communication libraries, message packages, command line tools, etc...
```bash
$ sudo apt install ros-foxy-ros-base
```
or we can install the base + RViz, demos and tutorials,
```bash
$ sudo apt install ros-foxy-desktop
```

You can pick any depending on your needs.
If you are not sure which to pick,
I would recommend you install the desktop version
in order to have all the tools you may need already installed.
However, note that if you intend to use some graphical applications
in your container, you have to take an extra step and set up some parameters
for your container.
All of this is detailed in the [aforementioned LXD post][LXD-post-gui].

Since the container was created especially for Foxy,
we will automatically source it in our '.bashrc',
```bash
$ echo "source /opt/ros/foxy/setup.bash" >> ~/.bashrc
```
Every times we will log into our container,
ROS 2 Foxy will be sourced and we will be ready to develop!

At last, we can install the Python package 'argcomplete' to enable
autocompletion for the ROS 2 command line tools.
This is totally optional, but also totally recommended:
```bash
sudo apt install python3-argcomplete
```

With Foxy installed, all there is left to do is to take it for a spin.

# Quick test

We will try the simple talker-listener demo,
mixing cpp and Python to make sure that
the installation went fine and that we can start developing right away.
Note that if you installed the 'base' version in the previous section,
you will need to install the following packages,
```bash
sudo apt install ros-foxy-demo-nodes-cpp ros-foxy-demo-nodes-py
```

For the test, let us start fresh and close our current shell.
We will then open 2 new ones, one for the publisher and one for the subscriber.

To start the publisher in the first shell enter,
```bash
$ ros2 run demo_nodes_cpp talker
[INFO] [1591461939.792327469] [talker]: Publishing: 'Hello World: 1'
[INFO] [1591461940.792228229] [talker]: Publishing: 'Hello World: 2'
[INFO] [1591461941.792184798] [talker]: Publishing: 'Hello World: 3'
...
```
and we can see that it starts publishing messages right away.

To start the listener in the second shell enter,
```bash
$ ros2 run demo_nodes_py listener
[INFO] [1591461964.793113956] [listener]: I heard: [Hello World: 5]
[INFO] [1591461965.792782570] [listener]: I heard: [Hello World: 6]
[INFO] [1591461966.792823099] [listener]: I heard: [Hello World: 7]
...
```
and we can see that it receives messages right away as well.

We are all set!

[//]: # (URLs)

[20-04]: https://ubuntu.com/blog/ubuntu-20-04-lts-arrives
[discourse-post]: https://discourse.ros.org/t/ros-foxy-fitzroy-released/14495
[rep2000]: https://www.ros.org/reps/rep-2000.html#foxy-fitzroy-may-2020-may-2023
[tuto]: https://index.ros.org/doc/ros2/Tutorials/
[release-page]: https://index.ros.org/doc/ros2/Releases/Release-Foxy-Fitzroy/
[LXD]: https://linuxcontainers.org/lxd/introduction/
[LXD-post]: /post/2020/lxc
[LXD-post-alias]: /post/2020/lxc#lxc-aliases-to-the-rescue
[LXD-post-gui]: /post/2020/lxc#using-graphical-applications
[foxy-install]: https://index.ros.org/doc/ros2/Installation/Foxy/
