---
layout: post
title: ROS Noetic development workflow in LXC
subtitle: Containerize all the things!
date: "2020-05-09T00:00:00Z"

reading_time: true
share: true
profile: false
comments: false
tags: [tutorial, LXC, LXD, ROS, Noetic, Ubuntu]
---

In this post, we will discuss how to setup a [Linux container][LXC]
\- a.k.a [LXC][LXC] - for our [ROS Noetic][noetic] development.
Developing in containers has several
advantages such as:
- allowing us to use a different Linux distribution than the one we've installed on
our host machine
- providing a repeatable course of actions
- messing around, installing a tons of dependencies without polluting our computer
- burning the container to the ground and starting fresh again easily

There are of course many other upsides but those are the one we are really
interested in for now.
We will see first how to get started with LXC and install the latest ROS release Noetic.
We will then configure our container so that it is able to share a workspace
with our host machine. We will also enable the use of graphical applications
from the container (e.g. Rviz, Gazebo).

The main prerequisites for this post are to be familiar with:
- the command terminal in Linux
- ROS development
- LXC

Note that I will be linking resources throughout the text,
make sure to check them whenever you need further information.

Finally, while we will be focusing on the latest ROS Noetic release,
the setup presented here applies not only to other ROS distributions
but likely to most projects, be them ROS-based or not.

---
# Content
*   [Setting up the LXC](#setting-up-the-lxc)
    *   [LXC aliases to the rescue](#lxc-aliases-to-the-rescue)
*   [Install ROS Noetic](#install-ros-noetic)
*   [Mounting a local workspace](#mounting-a-local-workspace)
*   [Using graphical applications](#using-graphical-applications)
    *   [Creating a LXD profile](#creating-a-lxd-profile)
    *   [Dedicated graphic card](#dedicated-graphic-card)
*   [Profile all the things!](#profile-all-the-things)
*   [Wrapping up](#wrapping-up)
---

# Setting up the LXC

We will start by installing LXD, a lightweight container hypervisor which
extends LXC functionality over the network.
LXD uses LXC under the covers for some container management tasks and
provides the 'lxc' command line interface tool we will use throughout this post.
For more information, you can refer to the
[LXC][lxc-doc] and [LXD][lxd-doc] documentation
on the Ubuntu website.

Alright, let us install LXD as a [snap][snapcraft-io] to make sure we always run
the most up to date stable version:

```bash
$ sudo snap install lxd
```

Before we can create our first container, we must initialize LXD,
```bash
$ sudo lxd init
```
This command will prompt you with a bunch of questions to fine tune LXD use.
Unless you know what you are doing, you can safely hit the default answers.

Finally, we will add our user to the 'lxd' group so that we can run lxd commands
without sudo,
```bash
$ sudo gpasswd -a "${USER}" lxd
```
You should log out and log in again for this to take effect.

## Creating the container

To create a new container, we will use the following command,
```bash
$ lxc launch {remote}:{image} {container-name}
```

Since Noetic runs on Ubuntu 20.04, we will fetch a Ubuntu 20.04 image
from the official Ubuntu remote,
```bash
$ lxc launch ubuntu:20.04 ros-noetic
```
We can check that the container was properly created and launched,
```bash
$ lxc list
+---------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
|     NAME      |  STATE  |         IPV4          |                     IPV6                      |   TYPE    | SNAPSHOTS |
+---------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
| ros-noetic    | RUNNING | 10.160.218.172 (eth0) | dd42:5ke1:fr68:2ca4:236:eff3:fe3r:7c21 (eth0) | CONTAINER | 0         |
+---------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
```
With our container up and running, we can open a shell in it with a non-root user
with the following command,
```bash
$ lxc exec ros-noetic -- sudo --login --user ubuntu
```
I know, this command is not very pretty nor easy to remember.
But worry not, we will create an alias to ease future use.

## LXC aliases to the rescue

LXC aliases, just like bash aliases, allow use to create a new CLI
keywords to which we can associate an action.
The command to create a new alias is,

```bash
$ lxc alias add {alias} '{command}'
```

As an example, let us create a shorter version of the `lxc list` command
that also prints a more compact result:

```bash
$ lxc alias add ls 'list --format csv -c n'
```
We can check that the alias is correctly created,

```bash
$ lxc alias list
+--------+----------------------------------------------------------------------------------+
| ALIAS  |                                      TARGET                                      |
+--------+----------------------------------------------------------------------------------+
| ls     | list --format csv -c n                                                           |
+--------+----------------------------------------------------------------------------------+
```

And we can now simply use it,
```bash
$ lxc ls
ros-noetic
```
That's pretty neat.

But our main goal with aliases was to simplify our shell
login to the container, so let's just do that.
Borrowing from the excellent [blog post by Simos Xenitellis][lxc-alias-blog]
about LXC aliases, we will create a new alias 'ubuntu' such as,
```bash
$ lxc alias add ubuntu 'exec @ARGS@ --mode interactive -- /bin/sh -xac $@ubuntu - exec /bin/login -p -f '
```
This alias allows us now to simply connect to our container with,
```bash
$ lxc ubuntu ros-noetic
```
That's much better isn't it?

# Install ROS Noetic

[ROS Noetic][noetic] is the latest and final ROS 1 release.
The ROS project hasn't come to an end, on the contrary, it rather look forward
and focus its efforts
toward the second version, namely ROS 2.
Nevertheless, ROS Noetic is an important release because it targets
Ubuntu 20.04, has official Python 3 support and will be supported until
May 2025 (more information on [Noetic wiki page][noetic-wiki]).
That leaves us plenty of time to learn and move to ROS 2.

To install it, let's first connect to our container using our new LXC alias,
<!-- we will simply follow the [official documentation][noetic-install]. -->

<!-- Let us execute a shell in our container using our new LXC alias, -->
```bash
$ lxc ubuntu ros-noetic
```

First, we will add the ROS packages repository to our sources.
Starting with the key,
```bash
$ apt-key adv --fetch-keys https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc
```
then the repository,
```bash
$ apt-add-repository http://packages.ros.org/ros/ubuntu
```

In case of trouble, you can also refer to the [official documentation][noetic-install].

We are all set to install ROS Noetic!

Here we can choose either of three installations;
we can install only the core components,
```bash
$ sudo apt install ros-noetic-ros-base
```
or core + the visualization stack (e.g. Rviz),
```bash
$ sudo apt install ros-noetic-desktop
```
or core + the visualization + simulation stacks (e.g. Gazebo),
```bash
$ sudo apt install ros-noetic-desktop-full
```
You can pick any depending on your needs.
If you are not sure, I would recommend you install only the core components
and later install other packages on a per-need basis:
```bash
$ sudo apt install ros-noetic-ros-base
...
$ sudo apt install ros-noetic-<package-I-need>
```
Simply to keep the size of the container as small as possible.

Finally, we will automatically source Noetic since this container is dedicated
to it,
```bash
$ echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
```
Every times we will log into our container, ROS Noetic will be sourced
and we will be ready to develop!

# Mounting a local workspace

What would our development workflow look like without some actual source code to
work on? Well, let us set up our ROS workspace.

Rather than copying/creating our workspace in the container,
we will keep it on the host machine. By doing so,
not only the workspace will survive deleting the LXC (persistence)
but we will also be able to share it across several LXC
thus across several ROS distros.

Hereafter, we will assume our workspace to be simply `~/workspace`
on the host with the classic tree,
```bash
$ tree ~/workspace
/home/user/workspace/
└── src
    └── my_ros_package
        └...
```

To share a folder with the container, we have to add a 'device disk' to it.
The general command to do so is of the form,

```bash
$ lxc config device add {container} {device-name} disk source={full-path-to-folder} path={full-path-inside-container}
```
filling up the placeholders for our use case, it reads,
```bash
$ lxc config device add ros-noetic workspace disk source=~/workspace path=/home/ubuntu/workspace
```
Once the device added, we have to configure the access rights so that we can read and write
the folder and its content in the container,
```bash
$ lxc config set ros-noetic raw.idmap "both $(id -u) $(id -g)"
```
We now have to restart the container for the changes to take effects,
```bash
$ lxc restart ros-noetic
```
Let us log back into our container,
```bash
$ lxc ubuntu ros-noetic
```
and verify that the folder is properly mounted,
```bash
$ ls -l
drwxr-xr-x 22 ubuntu ubuntu 4096 May 22 21:21 workspace
```
Looks like we are good!

In this section we have configured our container through the `lxc config`
cli tool. Note that container configuration is saved in a yaml file, which you
can review with,
```bash
$ lxc config show {container}
```
and directly edit with,
```bash
$ lxc config edit {container}
```
More on that later.

# Using graphical applications

This part is totally *optional* and depends on whether you are planning to run
some graphical applications (e.g. Rviz, Gazebo) in your container or not.
If you are not interested in running any gui in your container,
you may still want to have a quick look before jumping at the
'[Profile all the things!](#profile-all-the-things)' section.
If you do want to run graphical applications,
then we have to configure the container to support that.

Unlike in the previous section, we are not going to use the `lxc config` tool
to configure our container. Instead, we will introduce `lxc profile` as a way
to create easily reusable configurations.A *profile* is a set of parameters
that can be applied to a container in one go. It can describe a full fledged
setup or a particular feature as in our case below.
Furthermore a profile can be use by a single container or many. Reusability!

## Creating a LXD profile

Let us first create a profile named `gui`,
```bash
$ lxc profile create gui
```
we can now edit the profile,
```bash
$ lxc profile edit gui
```
and paste the following,
```yaml
config:
  environment.DISPLAY: :0
  raw.idmap: both 1000 1000
description: Enables graphical apps use.
devices:
  X0:
    path: /tmp/.X11-unix/X0
    source: /tmp/.X11-unix/X0
    type: disk
  mygpu:
    type: gpu
name: gui
used_by: []
```

Alternatively, you can use the following one liner,
```bash
curl https://gist.githubusercontent.com/artivis/37c961e157e99f6fcaff0204a0f59731/raw/ca4abd1a3c6b1d8a74910207903ac7723685dce1/gui.yaml | lxc profile edit gui
```

In this profile, there might be a couple things for you to tweak depending on
your machine. For instance your user id and guid,
```bash
raw.idmap: both 1000 1000
```
which you can retrieve respectively with:
```bash
$ id -u
1000
$ id -g
1000
```
You may also have to check your graphic card in use looking at the directory
`/tmp/.X11-unix/`.

Now that our profile is set up, we have to add it to our container,
```bash
lxc profile add ros-noetic gui
```
As previously, we have to restart the container for those change to take effect,
```bash
lxc restart ros-noetic
```

Alright, let us try to open Rviz to make sure everything went fine.
Open two shells to the container, one running the roscore and the second
running Rviz:

Shell 1
```bash
$ roscore
```
Shell 2
```bash
$ rosrun rviz rviz
```
We are getting really close to our regular development experience aren't we?

## Dedicated graphic card

If you have a dedicated graphic card on your host machine,
you will also have to install the *very same driver* in the container
in order to use graphical applications.
If you have an Nvidia card, the following should help you.
To figure out the driver version on the host we'll type,
```bash
$ nvidia-smi
Mon May 12 11:59:59 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.82       Driver Version: 440.82       CUDA Version: 10.2     |
+-------------------------------+----------------------+----------------------+
```
All we have to do now is to install the same driver in the container,
```bash
$ sudo apt install nvidia-440
```

# Profile all the things

We have seen in section ['Creating a LXD profile'](#creating-a-lxd-profile) how to
create a LXC profile to easily support running graphical
apps in our container(s).
As we mentioned before, a profile really only is a set of configurations
for our container.
So one may ask
> can't we create some other profiles to further group all the configs we've seen?

Well, yes we can! And guess what? Containers can have several profiles!
So we could totally create another profile to automatically
add the ROS apt repository, both for ROS 1 and ROS 2 respectively:

```bash
$ lxc profile create ros-apt
$ lxc profile edit ros-apt
```
and add,
```yaml
config:
  user.user-data: |
    #cloud-config
    runcmd:
      - "apt-key adv --fetch-keys https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc"
      - "apt-add-repository http://packages.ros.org/ros/ubuntu"
      - "apt-add-repository http://packages.ros.org/ros2/ubuntu"
description: "Add ROS apt repository"
devices: {}
name: ros
used_by: {}
```

Similarly we could create another profile to easily share our ROS workspace
as well,

```bash
$ lxc profile create ros-ws
$ lxc profile edit ros-ws
```
and add,
```yaml
config:
  raw.idmap: both 1000 1000
description: "Share the ROS workspace"
devices:
  workspaces:
    path: /home/ubuntu/workspace
    source: /home/user/workspace
    type: disk
name: ros-ws
used_by: {}
```
And remember to tweak both profiles (user id/guid etc.).

Both profiles will greatly help when creating a new container.
However, before we get all excited, let me tell you that
we have to be cautious when using them.
The reason is that both should be added a rather specific times
of the container creation. Let us see when that is.

First, the 'ros-apt' profile makes use of [cloud-init][cloud-init]
to preconfigure the container meaning that our `apt-key/apt-add-repository`
command will be run **only once** when the container is **first created**
(see [this other blog post by Simos Xenitellis][lxc-cloud-init-blog]
for more info about cloud-init in LXD).
To create a container with given profile(s),
the `lxc launch` commands changes to,
```bash
$ lxc launch --profile {profile-a} --profile {profile-b} {remote}:{image} {container-name}
```
which in our case looks like,
```bash
$ lxc launch --profile default --profile ros-apt ubuntu:20.04 ros-noetic2
```
Let me insist again. If you try to *add* the 'ros-apt' profile after the container
was created, *nothing will happen*:
~~`lxd profile add ros-noetic ros-apt`~~!

Concerning our 'ros-ws' profile, it is a bit of the opposite situation.
Indeed, when creating the container, a whole bunch of things are ran before
the 'ubuntu' user is set up. Since we are linking our workspace to `/home/ubuntu/`
we may arrive to early so to speak and it results in messing up the
proper set up of the user. For this profile, we therefore
*have to add it after the container creation*
(~~`lxc launch --profile ros-ws {remote}:{image} {container-name}`~~).

We can add our ros-ws profile to a container with,
```bash
$ lxc profile add ros-noetic ros-ws
```

This whole tempo story sounds annoying.
Alright let's call it a day and summarize how to set up a new container.

# Wrapping up

Well, that was quite a journey in LXD realm.
But our efforts were not vain for we have learned a lot
about LXD and set up some great tools.

Soon, the [ROS 2 Foxy][foxy] distro will be released
(5th of June).
How will we then create a Foxy container?
Well, that's quite simple now:

```bash
$ lxc launch --profile default --profile ros-apt --profile gui ubuntu:20.04 ros-foxy
$ lxc profile add ros-foxy ros-ws
$ lxc ubuntu ros-foxy
...
$ sudo apt install ros-foxy-desktop
```

Off we go!

<!--
# Speed up new LXC set up
@todo: snapshots + create container from snapshots + lxc-this script. Maybe for another post?
-->

<!-- Links -->

[//]: # (URLs)

[LXC]: https://linuxcontainers.org/
[lxc-started]: https://linuxcontainers.org/fr/lxc/getting-started/
[lxc-cheat-sheet]: https://medium.com/@tcij1013/lxc-lxd-cheetsheet-effb5389922d

[lxc-doc]: https://ubuntu.com/server/docs/containers-lxc
[lxd-doc]: https://ubuntu.com/server/docs/containers-lxd

[snapcraft-io]: https://snapcraft.io/

[noetic]: http://wiki.ros.org/noetic
[noetic-install]: http://wiki.ros.org/noetic/Installation/Ubuntu
[noetic-wiki]: http://wiki.ros.org/noetic

[foxy]: https://index.ros.org/doc/ros2/Releases/Release-Foxy-Fitzroy/

[lxc-alias-blog]: https://blog.simos.info/using-command-aliases-in-lxd-to-exec-a-shell/
[lxc-cloud-init-blog]: https://blog.simos.info/how-to-preconfigure-lxd-containers-with-cloud-init/

[cloud-init]: https://cloudinit.readthedocs.io/en/latest/index.html

[homesick]: https://github.com/technicalpickles/homesick
[homeshick]: https://github.com/andsens/homeshick
