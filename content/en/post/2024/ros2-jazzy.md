---
layout: post
title: Setting up ROS 2 Jazzy with LXD
subtitle: Down the (Jack)rabbit hole
date: "2024-05-31T00:00:00Z"

tags: [tutorial, ROS 2, Jazzy, LXC, LXD]
---

[ROS 2 Jazzy Jalisco is out!][ros2-jazzy-announ]

![ROS 2 Jazzy Jalisco logo](/post/2024/JazzyJalisco.png)

ROS 2 Release Team did it again,
shipping a brand new ROS 2 LTS distro.
The latest release is named Jazzy Jalisco,
a fine name given that the latest ROSCon (2023) was held in New Orleans!

It goes without saying that this is a collective effort that once again shows the solidity and the maturity of the ROS 2 community.

Being an LTS, ROS 2 Jazzy is based upon [Ubuntu 24.04][ubuntu-24-04], a.k.a "Noble Numbat", which is an LTS as well and was release just a month before Jazzy.

All of this great software being so recent, chances are,
you haven't upgraded yet.
So in this post we will see how to set it up a virtual environment so that we can start working right away.

If you're a long time reader,
this post should remind you of my ["ROS 2 Humble post"](/post/2022/ros2-humble).
Whether you've read it or not,
I'd encourage you to do so since this Jazzy post is gonna be short and referring a lot to the Humble one.

## Launching a Jazzy virtual environment

We'll see here how to quickly spawn a ROS 2 Jazzy virtual environment using [LXD][lxd].

Since [LXD][lxd] installation and initial setup is covered in many previous posts (e.g. [here](/post/2022/ros2-humble)),
we will assume here that it is ready to use.

From there,
we can immediately launch a Jazzy container using a ROS 2 [Cloud-init][cloud-init-doc] configuration I've put together (which you can [find on GitHub gist][ros2-cloud-init]):

```bash
lxc launch ubuntu:24.04 jazzy-lxc --config=user.user-data="$(curl -L https://gist.githubusercontent.com/artivis/0357fe03a5ae459bee8c55823fbb0af8/raw/ros2.cloudinit.yaml)"
```

This Cloud-init configuration will:

- set up the ROS 2 ppa
- install the `ros-base` metapackage as well as Colcon and rosdep
- initialise and update rosdep
- source the ROS 2 workspace

Everything we need to get started!

> The Cloudinit config works for any ROS 2 distro.
  Simply pick the Ubuntu version at launch and the corresponding ROS 2 distro will be set up (E.g. `launch 22.04` will spawn an Humble ready container)

As usual, we should wait for the container to be fully initialised:

```bash
$ lxc exec jazzy-lxc -- cloud-init status --wait
...............................................................................
status: done
```

Once done, we can now open a shell into the container as the default `ubuntu` user,

```bash
$ lxc exec jazzy-lxc -- sudo --login --user ubuntu
ubuntu@jazzy-lxc:~$ echo $ROS_DISTRO
jazzy
```

We're all set up.

## Bonus: Develop like a pro

While the following has been already covered in previous posts,
I think it is important enough that I should recall it here.
We are about to setup VSCode so that we can develop directly and seamlessly inside our LXD container.

To do so, we shall first add our public SSH key to the authorised keys in the container.
We can either copy the local key to the container:

```bash
cp ~/.ssh/id_rsa.pub /tmp/authorized_keys
lxc file push /tmp/authorized_keys jazzy-lxc/home/ubuntu/.ssh/authorized_keys -p
```

Or copy it from GitHub if you've set it up,

```bash
ubuntu@jazzy-lxc:~$ curl https://github.com/<user>.keys | tee -a ~/.ssh/authorized_keys
```

With the key set up, we can then access our container through SSH:

```bash
ssh ubuntu@<container-ip>
```

SSH'ing to our container is pretty neat but it's really a mean to an end.
What we really are after is the  [VS Code Remote - SSH extension][code-remote-ssh].
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

ROS 2 Jazzy Jalisco is an exciting new release and we don't have to wait to upgrade our machine to Ubuntu 24.04 to get started with it.

If you want to know more about all the new shiny thingies that Jazzy brings,
have a look at this [release page][jazzy-release] and/or jump directly into the [documentation][jazzy-doc].

I'd also strongly encourage you to revisit some older posts,
especially the following two that cover some interesting aspects of day-to-day development with LXD:

- ["Managing dotfiles"][container-home] - details how to manage our dotfiles and how that streamlines development in containers.
- ["ROS Noetic development workflow in LXC"][LXD-post] - is a previous iteration over this very post and also covers how to enable graphical applications in LXD.
  Something you may be interested in if you'd like to run things like Rviz!

Have fun !

[//]: # (URLs)

[ros2-jazzy-announ]: https://discourse.ros.org/t/ros-2-jazzy-jalisco-released/37862
[ubuntu-24-04]: https://releases.ubuntu.com/noble/
[jazzy-release]: https://docs.ros.org/en/jazzy/Releases/Release-Jazzy-Jalisco.html
[jazzy-doc]: https://docs.ros.org/en/jazzy/
[lxd]: https://linuxcontainers.org/lxd/introduction/
[cloud-init-doc]: https://cloudinit.readthedocs.io/en/latest/index.html
[ros2-cloud-init]: https://gist.github.com/artivis/0357fe03a5ae459bee8c55823fbb0af8
[code-remote-ssh]: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh

[container-home]: /post/2020/dotfiles
[LXD-post]: /post/2020/lxc
