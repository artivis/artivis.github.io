---
layout: post
title: How to build a ROS 2 Humble snap
subtitle: Deploy in a snap
date: "2022-05-23T00:00:00Z"

tags: [tutorial, ROS 2, Humble, snap]

draft: true
---

Earlier this week we celebrated the release of [ROS 2 Humble Hawksbill][lxd-ros2-humble] with a post detailing how to get started developing for the new release in containers.
This week we are tackling the logical next step in software development: packaging.
Indeed, once we're done developing our super cool ROS 2 Humble application,
we still have to get it out to the hands of our users.

In this post, we are going to see how to package a ROS 2 Humble application as a [snap][snap-about] with an 'hello world'-like example.

---

## Content

- [Content](#content)
- [A ROS 2 Humble snap you said?](#a-ros-2-humble-snap-you-said)
- [Setting up snapcraft](#setting-up-snapcraft)
- [A talker-listener example](#a-talker-listener-example)
  - [The snapcraft.yaml file](#the-snapcraftyaml-file)
  - [The ROS 2 Humble snap](#the-ros-2-humble-snap)
- [What's next?](#whats-next)

---

## A ROS 2 Humble snap you said?

> Snaps are app packages for desktop, cloud and IoT that are easy to install, secure, cross‐platform and dependency‐free.

## Setting up snapcraft

First thing first, let's get the tool that allows us to create snaps: [snapcraft][snapcraft].

```bash
sudo snap install --edge --classic snapcraft
```

Note that we specify here the `edge` release since ROS 2 Humble support is fresh out of the factory.

We are actually now all set up to package our first example.

## A talker-listener example

For this example, we'll take it easy as we won't even need to write any code.
Instead, we will package one of the [ROS 2 demo from the GitHub repository][ros2-ex].
I've picked a demo from the [`demo_nodes_cpp`][ros2-demo-node] package and more specifically the [`talker_listener.launch.py`][ros2-demo-launch] demo.

As its name suggests, this demo starts a talker which publishes a string message on the ROS 2 topic together with a listener which reads the said message.
Both print the message sent/received to the console to easily follow along.
And of course both are launched at once from a single launch file.

While this example may seem a little too simple,
it is a great entry door for a first contact with snaps.
We will see how easy it is to package ROS 2 applications with snaps.

### The snapcraft.yaml file

Snapcraft relies on the `snapcraft.yaml` configuration file to drive the packaging process.

Let us create one,

```bash
mkdir -p ~/ros2_ws/first_snap/
cd ~/ros2_ws/first_snap/
touch snapcraft.yaml
```

and populate it as follows,

```yaml
name: ros2-humble-talker-listener
version: '0.1'
summary: ROS2 Humble talker/listener example
description: |
  This example launches a ROS2 Humble talker and listener.

base: core22
confinement: strict

apps:
  ros2-humble-talker-listener:
    command: opt/ros/humble/bin/ros2 launch demo_nodes_cpp talker_listener.launch.py
    plugs: [network, network-bind]
    extensions: [ros2-humble]

parts:
  ros-demos:
    plugin: colcon
    source: https://github.com/ros2/demos.git
    source-branch: humble
    source-subdir: demo_nodes_cpp
    stage-packages: [ros-humble-ros2launch]
```

Believe it or not, this is all we need to create our snap.
Let us inspect this file more closely.

We can identify 3 distinct blocks:

At the top of the file there is some boiler-plate that is common to most snaps.
The snap `name`, `version` etc. Nothing unusual.
Then comes `base: core22` which is a base snap that will provide a runtime environment based on Ubuntu 22.04 for our application.
The last entry in this block is `confinement: strict` which states that our application is strictly confined and cannot access any resource on the host machine.

The second block, `apps`, specifies the application(s) that's exposed from the snap.
In our case, a single application is listed, whose `command` is very familiar.
Our application also lists some `interfaces` in the `plugs` section.
[Interfaces][snap-interfaces] allow our confined application to access specific resources of the host machine.
In this case, our snap will have access to network-related interfaces that allow for the ROS 2 topics to flow.
Finally, the application uses an `extensions` which automatically fills up some other fields which are common to ROS 2 snaps,
saving you some time and boiler-plate code.
If you are curious about what an extension does,
note that you can 'expand' it and reveal all of its secrets by issuing the following command,

```bash
snapcraft expand-extensions
```

We are now taking a look at the third block: `parts`.
The `parts` tag defines the different pieces that make up our final application.
They include a `source` entry for the source code.
This can be local files, a tarball, or as in this example, a GitHub repository, possibly at a specific branch.
A 'part' can include `build-packages` which are only required at build time and/or `stage-packages` which are needed at runtime.
Most importantly, each part is handled by a `plugin`.
Here we are using the `colcon` plugin, which makes use of the familiar build tool in ROS 2.
It too has its own options, which you can review with the command,

```bash
snapcraft help colcon
```

You can find further information about the [global metadata][snap-meta-data],
the [parts][snap-parts] and [their own metadata][snap-parts-meta],
the [colcon plugin][snap-colcon], [extensions][snap-extensions],
and even more details in the [documentation][snap-doc].

But for now, with our `snapcraft.yaml` file defined, it is time to build the snap.

### The ROS 2 Humble snap

To build the snap, issue the command `snapcraft` in a terminal,

```bash
$ snapcraft
Starting Snapcraft
...
[metalic sound of software being forged]
...
Created snap package
```

Snap, reveal yourself,

```bash
$ ls
ros2-humble-talker-listener_0.1_amd64.snap  snapcraft.yaml
```

There it is!
Alright, let's install it,

```bash
$ sudo snap install --dangerous ros2-humble-talker-listener_0.1_amd64.snap

ros2-humble-talker-listener 0.1 installed
```

Note the use of the `--dangerous` flag since we are installing a snap from disk instead of using the [store][snap-store].

Ok, but does it ~~bite~~ work?

```bash
$ ros2-humble-talker-listener
[INFO] [launch]: All log files can be found below /home/ubuntu/snap/ros2-humble-talker-listener/x6/ros/log/2022-05-24-15-24-50-823207-localhost-24895
[INFO] [launch]: Default logging verbosity is set to INFO
[INFO] [talker-1]: process started with pid [24944]
[INFO] [listener-2]: process started with pid [24946]
[talker-1] 2022-05-24 15:24:51.341 [RTPS_TRANSPORT_SHM Error] Failed to create segment ac1bbb6c86049e8a: Permission denied -> Function compute_per_allocation_extra_size
[listener-2] 2022-05-24 15:24:51.346 [RTPS_TRANSPORT_SHM Error] Failed to create segment e75bbd4ac39608ab: Permission denied -> Function compute_per_allocation_extra_size
[talker-1] 2022-05-24 15:24:51.348 [RTPS_MSG_OUT Error] Permission denied -> Function init
[listener-2] 2022-05-24 15:24:51.348 [RTPS_MSG_OUT Error] Permission denied -> Function init
[talker-1] [INFO] [1653420292.379305200] [talker]: Publishing: 'Hello World: 1'
[listener-2] [INFO] [1653420292.380423139] [listener]: I heard: [Hello World: 1]
[talker-1] [INFO] [1653420293.379072149] [talker]: Publishing: 'Hello World: 2'
[listener-2] [INFO] [1653420293.379286698] [listener]: I heard: [Hello World: 2]
[talker-1] [INFO] [1653420294.379120962] [talker]: Publishing: 'Hello World: 3'
[listener-2] [INFO] [1653420294.379682258] [listener]: I heard: [Hello World: 3]
[talker-1] [INFO] [1653420295.379087916] [talker]: Publishing: 'Hello World: 4'
[listener-2] [INFO] [1653420295.379301853] [listener]: I heard: [Hello World: 4]
[talker-1] [INFO] [1653420296.379133958] [talker]: Publishing: 'Hello World: 5'
[listener-2] [INFO] [1653420296.379475925] [listener]: I heard: [Hello World: 5]
```

Yes it does! How great.
What's even greater is that you could now install and run this snap on another computer even if it doesn't have ROS 2 installed!

> Note that the error message from FastDDS, the underlying DDS library,
does not prevent the application from running.
We can safely ignore that for now.
I may come back to the ins and outs of this error in a later post.

## What's next?

We've seen how to create a snap for a ROS 2 Humble application.
While this demo is fairly trivial,
packaging a more complex ROS stack isn't much more complicated.
To convince you, have a look at the series ["How to set up TurtleBot3 in minutes with snaps"][tb3-1] (part [one][tb3-1] & [two][tb3-2]) where I detail how to snap the entire Turtlebot3.

Furthermore, we've seen how you can effectively and easily package your ROS 2 application.
How about distributing it now?
Have a look at how to do so with the store [here][snap-release].

Finally, do feel free to ask any questions on the [snapcraft forums][snap-forum],
or on [ROS answers][ros-ans]. I’d love to hear any feedback you have.

[//]: # (URLs)

[ros2-humble-announ]: https://www.openrobotics.org/
[snapcraft]: https://snapcraft.io/
[lxd-ros2-humble]: /post/2022/ros2-humble
[ros2-ex]: https://github.com/ros2/demos
[ros2-demo-node]: https://github.com/ros2/demos/tree/master/demo_nodes_cpp
[ros2-demo-launch]: https://github.com/ros2/demos/blob/master/demo_nodes_cpp/launch/topics/talker_listener.launch.py
[snap-about]: https://snapcraft.io/docs/robotics
[snap-interfaces]: https://snapcraft.io/docs/supported-interfaces
[snap-meta-data]: https://snapcraft.io/docs/adding-global-metadata
[snap-parts]: https://snapcraft.io/docs/adding-parts
[snap-parts-meta]: https://snapcraft.io/docs/snapcraft-parts-metadata
[snap-colcon]: https://snapcraft.io/docs/the-colcon-plugin
[snap-extensions]: https://snapcraft.io/docs/supported-extensions
[snap-doc]: https://snapcraft.io/docs
[snap-store]: https://snapcraft.io/store
[snap-release]: https://snapcraft.io/docs/releasing-your-app
[tb3-1]: https://ubuntu.com/blog/how-to-set-up-turtlebot3-in-minutes-with-snaps
[tb3-2]: https://snapcraft.io/blog/how-to-set-up-turtlebot3-in-minutes-with-snaps-2
[snap-forum]: https://forum.snapcraft.io/
[ros-ans]: https://answers.ros.org/questions/