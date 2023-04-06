---
layout: post
title: Setting up ROS 2 Humble with LXD
subtitle: An Humble container
date: "2022-05-23T00:00:00Z"

tags: [tutorial, ROS 2, Humble, LXC, LXD]
---

[ROS 2 Humble Hawksbill is out!][ros2-humble-announ]

![ROS 2 Humble Hawksbill logo](/img/post/2022/HumbleHawksbill.png)

Congratulations to [Open Robotics][or] and through them to the entire ROS 2 community.
This is really quite an event as it marks an important milestone: it is the first 5 year LTS release for ROS 2.
The release of maturity as people say.

ROS 2 Humble officially supports Ubuntu 22.04, but chance is,
you haven't made that jump yet.
So in this post we will see how to set it up in an [LXD][lxd] container so that we can start ~~playing~~ working on our good old machine.

We've covered how to get started with LXD in numerous previous posts;
so here we will only go over the commands to get started as quickly and conveniently as possible.

## Setting up LXD

[LXD][lxd] is a manager for [LXC][lxc].

With this out of the way,
the simplest way to install it is to use the snap package:

```bash
sudo snap install lxd --channel=5.0/stable
```

Note that we are specifying the use of the recently released 5.0 LTS version.

Let us now configure LXD,

```bash
sudo lxd init --minimal
```

The minimal setup will configure LXD with default options.
You may find more information about this in the [online documentation][lxd-doc].

### Launching a first container

To make sure that everything works fine,
let us try to launch a first container rocking Ubuntu 22.04,

```bash
lxc launch ubuntu:22.04 first-container
```

Is it running?

```bash
$ lxc list
+---------------------+---------+----------------------+-----------------------------------------------+-----------+-----------+
|        NAME         |  STATE  |         IPV4         |                     IPV6                      |   TYPE    | SNAPSHOTS |
+---------------------+---------+----------------------+-----------------------------------------------+-----------+-----------+
| first-container     | RUNNING | 10.190.86.230 (eth0) | fd42:1726:4b4d:8cd3:216:3eff:fed8:7a69 (eth0) | CONTAINER | 0         |
+---------------------+---------+----------------------+-----------------------------------------------+-----------+-----------+
```

It seems so; then let's try to get a shell inside the container,

```bash
$ lxc shell first-container
root@ros2-humble:~ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description: Ubuntu 22.04 LTS
Release: 22.04
Codename: jammy
```

We're in, running 22.04.
Everything looks good!

From there we could simply follow the [installation instructions from the ROS 2 Humble documentation][humble-install].
But where is the fun in that?
Instead we will create a [LXD profile][lxd-profile] that will do all the lifting for us every time we will create a new container.

## An Humble profile

First we need to create a profile,

```bash
lxc profile create ros2-humble
```

We will then edit it to add the bits we need.
But rather than editing the profile itself,
we will work on a good-old yaml file, located on our machine.
The reason is that it is more convenient and most importantly,
we will be able to carry that yaml file and re-use it on other machines.
Let's create that profile,

```bash
mkdir -p ~/lxc_dotfiles/profile/
touch ~/lxc_dotfiles/profile/ros2-humble.yml
```

which we populate as follows,

```yaml
config:
  user.user-data: |
    #cloud-config
    # Add the ROS 2 sources
    apt:
      sources:
        ros2:
          source: "deb [arch=amd64] http://repo.ros2.org/ubuntu/main jammy main"
          keyid: C1CF 6E31 E6BA DE88 68B1 72B4 F42E D6FB AB17 C654
    package_upgrade: true
    packages:
      # dev
      - vim
      - silversearcher-ag
      # ROS 2 dev
      - locales
      - curl
      - gnupg
      - lsb-release
      - build-essential
      - cmake
      - git
      - python3-argcomplete
      - python3-colcon-common-extensions
      - python3-flake8
      - python3-flake8-blind-except
      - python3-flake8-builtins
      - python3-flake8-class-newline
      - python3-flake8-comprehensions
      - python3-flake8-deprecated
      - python3-flake8-docstrings
      - python3-flake8-import-order
      - python3-flake8-quotes
      - python3-pip
      - python3-pytest
      - python3-pytest-cov
      - python3-pytest-repeat
      - python3-pytest-rerunfailures
      - python3-rosdep
      - python3-setuptools
      - python3-vcstool
      - wget
      # ROS 2 packages
      - ros-humble-ros-core
    runcmd:
      # System setup
      - "locale-gen en_US en_US.UTF-8"
      - "update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8"
      - "export LANG=en_US.UTF-8"
      #
      # Fetch ROS 2 sources
      - [su, ubuntu, -c, "mkdir -p /home/ubuntu/ros2_ws/src"]
      - [su, ubuntu, -c, "wget https://raw.githubusercontent.com/ros2/ros2/master/ros2.repos -O /home/ubuntu/ros2_ws/ros2.repos"]
      - [su, ubuntu, -c, "vcs import /home/ubuntu/ros2_ws/src < /home/ubuntu/ros2_ws/ros2.repos"]
      #
      # Install deps
      - [su, ubuntu, -c, "sudo rosdep init"]
      - [su, ubuntu, -c, "rosdep update"]
      - [su, ubuntu, -c, "rosdep install --from-paths /home/ubuntu/ros2_ws/src --ignore-src -y --skip-keys 'fastcdr rti-connext-dds-6.0.1 urdfdom_headers gazebo_ros_pkgs' --rosdistro humble"]
    final_message: "ROS 2 Humble Hawksbill dev container ready!"
description: "A profile to automatically a ROS 2 Humble dev container."
devices: {}
name: ros2-humble
used_by: []
```

We will not go into the details of this profile,
note simply that it takes advantage of [LXD support of cloud-init][lxd-cloud-init] to set up the ppa, install packages and fetch the ROS 2 Humble source code.
Alright, let us carry on and launch our Humble container.
But let's not forget to edit our LXD profile from the yaml file we have just created,

```bash
lxc profile edit ros2-humble < ~/lxc_dotfiles/profile/ros2-humble.yml
```

### Launching a ROS 2 Humble container

With our profile ready,
all we have to do is to launch a new container with that profile,

```bash
$ lxc launch ubuntu:22.04 -p default -p ros2-humble ros2-humble
Creating ros2-humble
Starting ros2-humble
```

Note that we are also adding the `default` profile which sets up the root filesystem and the network of our container.

At this point the container is up and running.
However it isn't quite ready yet.
Indeed, it is crunching the specified commands to get our ROS 2 environment ready.

To monitor this process (the cloud-init initialization),
we can issue the following command,

```bash
$ lxc exec ros2-humble -- cloud-init status --wait
...............................................................................
status: done
```

which will print dots on the terminal as the initialization goes on,
and will do so until it is done.
When the command returns, the container is fully ready.

We can now open a shell into the container as the default `ubuntu` user,

```bash
$ lxc exec ros2-humble -- sudo --login --user ubuntu
ubuntu@ros2-humble:~$ ls
ros2_ws
ubuntu@ros2-humble:~$ ls /opt/ros/humble
bin  cmake  include  lib  local  local_setup.bash  local_setup.sh  _local_setup_util.py  local_setup.zsh  setup.bash  setup.sh  setup.zsh  share  src  tools
```

The source workspace is there and the packages are installed.
Looks like we are ready to develop.

### One-liniiiiing

Phew; that was quite a few commands.
Some we will definitely not remember.
Let's make our life easier and create some [LXD aliases][lxd-alias].

First, an alias to create a new ROS 2 Humble container given a name,

```bash
lxc alias add launch-ros2-humble 'launch ubuntu:22.04 -p default -p ros2-humble @ARGS@'
```

use e.g.,

```bash
lxc launch-ros2-humble another-ros2-humble
```

Then an alias to wait for a container to be ready,

```bash
lxc alias add wait-for 'exec @ARGS@ -- cloud-init status --wait'
```

use e.g.,

```bash
lxc wait-for another-ros2-humble
```

And one to open a shell as the ubuntu user,

```bash
lxc alias add ubuntu 'exec @ARGS@ --mode interactive -- /bin/sh -xac $@ubuntu - exec /bin/login -p -f'
```

use e.g.,

```bash
lxc ubuntu another-ros2-humble
```

Alright, we can now launch a branch new container,
wait for it to be ready and open a shell,

```bash
$ lxc launch-ros2-humble ros2-humble && lxc wait-for ros2-humble && lxc ubuntu ros2-humble
Creating ros2-humble
Starting ros2-humble
..................................................................................................................................................................................................................................................................................
status: done
+ exec /bin/login -p -f ubuntu
Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.4.0-110-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun May 22 10:06:39 PM UTC 2022

  System load:           1.06347656253
  Usage of /home:        unknown
  Memory usage:          45%
  Swap usage:            68%
  Temperature:           59.0 C
  Processes:             30
  Users logged in:       0
  IPv4 address for eth0: 10.190.86.215
  IPv6 address for eth0: fd42:1726:4b4d:8cd3:216:3eff:fefb:fc68

0 updates can be applied immediately.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

ubuntu@ros2-humble:~$
```

That was easy...

PS- you'll excuse me for abusing a little the one-liner claim :D
(altho we could hide that in a bash alias...).

## Develop like a pro

That was a nice ride and all but now what?
Well, we could enable an easy SSH access to our container.
To do so we will push our SSH key to the container,

```bash
cp ~/.ssh/id_rsa.pub /tmp/authorized_keys
lxc file push /tmp/authorized_keys ros2-humble/home/ubuntu/.ssh/authorized_keys -p
```

and we can then access our container through SSH,

```bash
ssh ubuntu@<container-ip>
```

We should really add this step to our profile instead.
And to do so we will simply add the following lines,

```yaml
config:
  user.user-data: |
    #cloud-config
    # Retrieve SSH key to give the default user SSH access to the container
    # The key here is retrieved from GitHub user 'myuser'
    ssh_import_id:
      - gh:myuser
    # Add the ROS 2 sources
    apt:
    ...
```

which will fetch our SSH public key from our GitHub user and set them up for the default user in the container.
If you prefer to retrieve the key from Launchpad instead,
replace `- gh:myuser` with `- lp:myuser`.
Also, make sure to replace `myuser` with your actual user; just saying :D.

Let's make sure we update our profile again,

```bash
lxc profile edit ros2-humble < ~/lxc_dotfiles/profile/ros2-humble.yml
```

Note that because cloud-init only runs during the first boot,
we would need to recreate our container for this to take effect,

```bash
lxc stop ros2-humble && lxc delete ros2-humble
lxc launch-ros2-humble ros2-humble && lxc wait-for ros2-humble
```

Again, once ready, we can SSH into our container,

```bash
ssh ubuntu@<container-ip>
```

### Using VS Code with LXD

Since we can SSH,
we can also make use of the [VS Code Remote - SSH extension][code-remote-ssh].
This neat little feature allows us to open any file or folder in a container,
a VM, or a remote machine.

To install the extension,
head to Code and search for "remote ssh" in the Extensions panel or type in your terminal,

```bash
code --install-extension ms-vscode-remote.remote-ssh
```

Then, open Code's Command Palette (`Ctrl+Shift+P`),
search for `Remote-SSH: Connect to Host...` and enter your SSH connection details:
`ubuntu@<container_ip>`.
From there you are able to browse and work on your fresh ROS 2 Humble workspace
using **File > Open...** or **File > Open Workspace...** just as you would locally!

## Conclusion

ROS 2 Humble Hawksbill is an exciting new release and I can't wait to get started with it.
Fortunately, as we've seen in this post,
we don't have to wait to upgrade our machine to Ubuntu 22.04 to do so.

If you want to know more about all the new shiny thingies that Humble brings,
have a look at this [release page][humble-release] and/or jump directly into the [documentation][humble-doc].

Let us have fun!

### Bonus

I'd strongly encourage you to revisit some older posts,
especially the following two that cover some interesting aspects of day-to-day development with LXD:

- ["Managing dotfiles"][container-home] - details how to manage our dotfiles and how that streamlines development in containers.
- ["ROS Noetic development workflow in LXC"][LXD-post] - is a previous iteration over this very post and also covers how to enable graphical applications in LXD.
  Something you may be interested in if you'd like to run things like Rviz!

[//]: # (URLs)

[ros2-humble-announ]: https://discourse.ros.org/t/ros-2-humble-hawksbill-released
[or]: https://www.openrobotics.org/
[humble-release]: https://docs.ros.org/en/foxy/Releases/Release-Humble-Hawksbill.html
[humble-doc]: https://docs.ros.org/en/humble/
[humble-install]: https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html
[lxc]: https://linuxcontainers.org/lxc/introduction/
[lxd]: https://linuxcontainers.org/lxd/introduction/
[lxd-doc]: https://linuxcontainers.org/lxd/getting-started-cli/#initial-configuration
[lxd-profile]: https://linuxcontainers.org/lxd/docs/master/profiles/
[lxd-cloud-init]: https://linuxcontainers.org/lxd/docs/master/cloud-init/
[lxd-alias]: https://linuxcontainers.org/lxd/advanced-guide/#command-aliases
[code-remote-ssh]: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh

[container-home]: /post/2020/dotfiles
[LXD-post]: /post/2020/lxc
