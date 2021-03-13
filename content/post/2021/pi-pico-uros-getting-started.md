---
layout: post
title: Getting started with micro-ROS on the Pi Pico
subtitle: On running ROS2 on microcontrollers
date: "2021-03-11T00:00:00Z"

reading_time: true  # Show estimated reading time?
share: true  # Show social sharing links?
profile: false  # Show author profile?
comments: false  # Show comments?
tags: [tutorial, Raspberry Pi, Pi Pico, ROS2, micro-ROS, VSCode]
---

In this post we will see how the Pi Pico can natively speak to a ROS2 graph using micro-ROS.
We will set up a project in VSCode, compile and upload it to the microcontroller.
We thus assume that you are somewhat familiar with [ROS2 development][ros2-doc] and [VSCode][VSCode].

---

## Content

- [What is this all about?](#what-is-this-all-about)
  - [The Raspberry Pi Pico](#the-raspberry-pi-pico)
  - [micro-ROS](#micro-ros)
- [Getting started](#getting-started)
  - [Installing dependencies](#installing-dependencies)
  - [Fetching the sources](#fetching-the-sources)
  - [Setting up VSCode](#setting-up-vscode)
- [Running the example](#running-the-example)
  - [Wait a minute. What does it do?](#wait-a-minute-what-does-it-do)
  - [Uploading to the Pi Pico](#uploading-to-the-pi-pico)
  - [Installing the micro-ros-agent](#installing-the-micro-ros-agent)
  - [Actually running the example](#actually-running-the-example)
- [What's next?](#whats-next)

---

## What is this all about?

### The Raspberry Pi Pico

The Raspberry Pi Pico, [announced in late January 2021][pico-anounce],
is the newest release of the Raspberry Pi Foundation which received a ton of attention (a quick search on Google and/or Youtube will convince you).
And that's for a good reason.
Compared to its well known predecessors,
this new board differs in two major ways:
it is an *in-house* designed open-hardware *microcontroller*!
Yes, the chip itself is designed by the Pi's engineers and it is fully open-hardware.
And as usually with the Pi foundation,
it is incredibly affordable at just 4$.

The details concerning the board itself,
the differences between microprocessor and microcontroller,
the 101 getting started or what can the Pi Pico do;
all of that is beyond the scope of this post.
But I strongly encourage you having a look for yourself,
whether you are familiar with microcontrollers or not.

### micro-ROS

In the ROS (1) realm, microcontrollers have always been sort of second class citizens.
They can't interact directly with the ROS graph and developers have to rely on libraries such as [rosserial][rosserial].
But ROS2 is a whole new world and things are changing.

> micro-ROS puts ROS 2 onto microcontrollers, making them first class participants of the ROS 2 environment.

The [micro-ROS][uros_website] project is an effort led by [big industrial names][uros_supports] such as Bosch,
eProsima, Fiware Foundation, notably through the [OFERA][ofera_website] H2020 project,
and a myriad of partners and collaborators including e.g. Amazon and Canonical.

So what is it? It is essentially a thin wrapper (see its [design document][uros_design_doc]) on top of 'DDS for eXtremely Resource Constrained Environments' ([DDS-XRCE][dds_xrce_specs]),
running on a real-time OS, allowing microcontrollers to 'speak' to a ROS2 graph (the usual talker/listener) using an optimized subset of the DDS protocol.
It relies on a 'bridged' communication architecture with a 'broker' named the ['micro-ros-agent'][uros_agent_github].
The agent is in charge of the interfacing between the ROS2 graph and one or several micro-ROS devices.

More details can be found on the [micro-ROS website][uros_website] including how it compares/differs from rosserial (see [here][uros_vs_other] and [here][uros_vs_rosserial]).

## Getting started

Alright, so now that we have clarified a couple terms,
let us get started, step by step,
with micro-ROS on Pi Pico with the [official example available on github][uros_pico_github].
Note that for this tutorial I am running Ubuntu 20.04 with the [VSCode snap][VSCode-snap].

If you are not running Ubuntu 20.04 yet, you could consider using a LXD container.
You can refer to my previous post ['ROS Noetic development workflow in LXC'][lxc-post] to help you get started setting up the container.

### Installing dependencies

Let's start simple by installing the couple necessary dependencies,

```bash
sudo apt install build-essential cmake gcc-arm-none-eabi libnewlib-arm-none-eabi doxygen git python3
```

### Fetching the sources

We will now create a workspace and fetch all the sources,

```bash
mkdir -p ~/micro_ros_ws/src
cd ~/micro_ros_ws/src
git clone --recurse-submodules https://github.com/raspberrypi/pico-sdk.git
git clone https://github.com/micro-ROS/micro_ros_raspberrypi_pico_sdk.git
```

The first repository is the Pi Pico SDK provided by the Pi foundation.
The second contains a precompiled micro-ROS stack together with a hello-world-like example.

### Setting up VSCode

Let us now open the example in VSCode and set it up.
To follow along, you will need two VSCode extensions that are rather common for C++ development.
These extensions are the [C++ extension][c++-extension] and [CMake tools][cmake-tools] for VSCode.
After installing them, we will create a configuration file for CMake tools and set a variable so that our project knows where to find the Pi Pico SDK.
To do so, simply type,

```bash
cd ~/micro_ros_ws/src/micro_ros_raspberrypi_pico_sdk
mkdir .vscode
touch .vscode/settings.json
```

Open the newly created file with your favorite editor,

```bash
vi .vscode/settings.json
```

and add the following,

```json
{
    "cmake.configureEnvironment": {
        "PICO_SDK_PATH": "/tmp/pico/src/pico-sdk",
    },
}
```

This variable is an environment variable that is only passed to CMake at configuration time.
See the [CMake-Tools documentation][cmake-tools-doc] for more info.

Let us now open it,

```bash
code .
```

Before running the CMake configuration and build it,
we must select the appropriate 'kit' (maybe VSCode has already asked you to do so).
Open the palette (ctrl+shift+p) and search for `'CMake: Scan for Kits'` and then `'CMake: Select a Kit'` and make sure to select the compiler we've installed above, that is `'GCC for arm-non-eabi'`.

We're all set, let us build the example!
Open the palette again and hit `'CMake: Build'`.

## Running the example

### Wait a minute. What does it do?

Right, let's break down very briefly what the example does.
It sets up a node called `'pico_node'`,
then a publisher publishing a `'std_msgs/msg/int32.h'` message on topic `'pico_publisher'`,
a recurring timer and an executor to orchestrate everything.
Every 0.1 second, the executor spins.
But only every second, the timer will have the publisher publish a message and increase the message data by 1.
Simple. So let's try it out.

### Uploading to the Pi Pico

If everything went fine during compilation,
you should see a new `'build'` folder in your project view.
In this folder, you will find the file that we should now upload to the Pi Pico,
it is named here `'pico_micro_ros_example.uf2'`.
To upload it, simply connect the board with a USB cable **while pressing** the tiny white button labelled `'BOOTSEL'`.
Doing so, the Pi Pico will mount similarly to a flash drive allowing us to very easily copy/paste the '`.uf2`' file.

Head to a terminal and type,

```bash
cd build
cp pico_micro_ros_example.uf2 /media/$USER/RPI-RP2
```

Once the file is copied,
the board will automatically reboot and start executing the example.

Easy-peasy.

### Installing the micro-ros-agent

We have seen in the introduction that micro-ROS has a bridged communication architecture.
We thus have to build that bridge.
Well, fortunately the development team has built it already and distributes it both as a [Snap][uros-agent-snap] or a [Docker image][uros-agent-docker].
Here we'll make use of the former.
If you are using Ubuntu 16.04 or later, snap is already pre-installed and ready to go.
If you are running another OS, you can either [install snap][install-snap] or make use of the Docker image.

To install the micro-ros-agent snap, type,

```bash
sudo snap install micro-ros-agent
```

After installing it, and because we are using a serial connection,
we need to configure a couple things.
First we need to enable the `'hotplug'` feature,

```bash
sudo snap set core experimental.hotplug=true
```

and restart the snap demon so that it takes effect,

```bash
sudo systemctl restart snapd
```

After making sure the Pi Pico is plugged, execute,

```bash
$ snap interface serial-port
name:    serial-port
summary: allows accessing a specific serial port
plugs:
  - micro-ros-agent
slots:
  - snapd:pico (allows accessing a specific serial port)
```

What we see here is that the micro-ros-agent snap has a serial '`plug`' while a '`pico`' `'slot'` magically appeared.
As per the semantic, we probably should connect them together. To do so run,

```bash
snap connect micro-ros-agent:serial-port snapd:pico
```

We are now all set to finally run our example.

### Actually running the example

With the Pi Pico plugged through USB,
we will start the micro-ros-agent as follows,

```bash
micro-ros-agent serial --dev /dev/ttyACM0 baudrate=115200
```

and wait a couple seconds for the Pi Pico's LED to light up indicating that the main loop is running.
In case it does not light up after a few long seconds (count up to 10 mississippi),
you may want to unplug/replug the board in order to reboot it.
The initialization procedure of the example lacks a few error checking.
Hey, could fixing that be **your** first project?

So now the LED should shine a bright green.
That's cool.
Do you know what's cooler?
Running on your host machine,

```bash
$ source /opt/ros/dashing/setup.bash
$ ros2 topic echo /pico_publisher
data: 41
---
data: 42
---
```

Awesome!

And hitting a

```bash
$ ros2 node list
/pico_node
```

proves that the micro-ROS node running on the Pi Pico is visible to ROS2 on the host machine.

Yatta!

## What's next?

For a long time it wasn't convenient to mix microcontrollers and ROS.
But this is about to seriously change as we've just seen.
No doubt that both micro-ROS and the Pi Pico will bolster great robotics applications (and more!).

In this tutorial we've reached a great starting point with a ROS2-based project ready to spin on the suppa-cool suppa-affordable Pi Pico.

Of course this wouldn't have been possible without the micro-ROS dev team and Cyberbotics engineer Darko Lukić ([@lukicdarkoo][lukicdarkoo]) who has put together the initial example we've just used.
As often, there are super smart people out there making complicated stuff very accessible,
shout out to them.

I'm personally going to keep playing with micro-ROS on Pi Pico,
first because it is fun and second because I have a couple ideas up my sleeves.
Be sure that if they become reality you'll hear about them on this blog.

What about you? Do you have some cool projects already in mind?

[//]: # (URLs)

[ros2-doc]: https://docs.ros.org/en/foxy/index.html
[rosserial]: http://wiki.ros.org/rosserial

[pico-anounce]: https://www.raspberrypi.org/blog/raspberry-pi-silicon-pico-now-on-sale/

<!-- uROS main links -->
[uros_website]: https://micro-ros.github.io/
[uros_supports]: https://micro.ros.org/docs/overview/users_and_clients/
[uros_design_doc]: https://micro-ros.github.io/docs/concepts/client_library/decision_paper/
[uros_vs_other]: https://micro-ros.github.io/docs/overview/comparison/
[uros_vs_rosserial]: https://micro.ros.org/docs/concepts/middleware/rosserial/
[uros_agent_github]: https://github.com/micro-ROS/micro-ROS-Agent
[uros-agent-docker]: https://hub.docker.com/r/microros/micro-ros-agent
[uros-agent-snap]: https://snapcraft.io/micro-ros-agent
[uros_pico_github]: https://github.com/micro-ROS/micro_ros_raspberrypi_pico_sdk

[dds_xrce_specs]: https://www.omg.org/spec/DDS-XRCE/
[ofera_website]: http://www.ofera.eu/

[lxc-post]: https://artivis.github.io/post/2020/lxc/

[VSCode-snap]:https://snapcraft.io/code
[VSCode]: https://code.visualstudio.com
[c++-extension]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools
[cmake-tools]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools

[install-snap]: https://snapcraft.io/docs/installing-snap-on-ubuntu

[lukicdarkoo]: https://github.com/lukicdarkoo