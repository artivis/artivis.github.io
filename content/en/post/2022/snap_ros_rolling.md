---
draft: true
---

# How to build a snap using ROS 2 Rolling

Recently, a member of the community ask me how to build a snap for [ROS 2 Galactic][ros2-galactic].
This is an interesting question,
not only because it give us the occasion to write about it but also because it incidentally gives us an occasion to look at snapcraft extensions.

In this post, we will see how to build a snap using [ROS 2 Rolling][ros2-rolling],
the rolling release distro of ROS 2 (hence the name).
Note that the very same principle applies to ROS 2 Galactic in case that is your targeted distro.

The following tutorial, and the example it uses,
is largely based on an earlier post "[How to build a snap using ROS 2 Foxy][foxy-snap]".
For this reason we will not cover the entire `snapcraft.yaml` file entries but only the bit that differ.
If you are new to snapcraft or need a brief reminder,
please do start with the aforementioned post before continuing here.

Alright, let us get started.

<!-- Let us start from a [ROS 2 Foxy example][foxy-snap]. -->

## Starting from a Foxy base

Let us start from the simple talker-listener example given in the post "[How to build a snap using ROS 2 Foxy][foxy-snap]".
To do so we will create the `snapcraft.yaml` file,

```bash
mkdir -p ~/ros2-rolling-snap && cd ~/ros2-rolling-snap
nano snapcraft.yaml
```

and populate it with the following,

```yaml
name: ros2-talker-listener
version: '0.1'
summary: ROS2 Talker/Listener Example
description: |
  This example launches a ROS2 talker and listener.

grade: devel
confinement: strict
base: core20

parts:
  ros-demos:
    plugin: colcon
    source: https://github.com/ros2/demos.git
    source-branch: foxy
    source-subdir: demo_nodes_cpp
    build-packages: [make, gcc, g++]
    stage-packages: [ros-foxy-ros2launch]

apps:
  ros2-talker-listener:
    command: opt/ros/foxy/bin/ros2 launch demo_nodes_cpp talker_listener.launch.py
    plugs: [network, network-bind]
    extensions: [ros2-foxy]
```

### Expanding the extension

```bash
snapcraft expand-extensions --enable-experimental-extensions
```

```bash
$ snapcraft expand-extensions --enable-experimental-extensions
*EXPERIMENTAL* extension 'ros2-foxy' enabled.
name: ros2-talker-listener
version: '0.1'
summary: ROS2 Talker/Listener Example
description: 'This example launches a ROS2 talker and listener.

  '
grade: devel
confinement: strict
base: core20
parts:
  ros-demos:
    plugin: colcon
    source: https://github.com/ros2/demos.git
    source-branch: foxy
    source-subdir: demo_nodes_cpp
    build-packages:
    - make
    - gcc
    - g++
    stage-packages:
    - ros-foxy-ros2launch
    build-environment:
    - ROS_VERSION: '2'
    - ROS_DISTRO: foxy
  ros2-foxy-extension:
    build-packages:
    - ros-foxy-ros-core
    override-build: install -D -m 0755 launch ${SNAPCRAFT_PART_INSTALL}/snap/command-chain/ros2-launch
    plugin: nil
    source: $SNAPCRAFT_EXTENSIONS_DIR/ros2
apps:
  ros2-talker-listener:
    command: opt/ros/foxy/bin/ros2 launch demo_nodes_cpp talker_listener.launch.py
    plugs:
    - network
    - network-bind
    command-chain:
    - snap/command-chain/ros2-launch
    environment:
      PYTHONPATH: $SNAP/opt/ros/foxy/lib/python3.8/site-packages:$SNAP/usr/lib/python3/dist-packages:${PYTHONPATH}
      ROS_DISTRO: foxy
      ROS_VERSION: '2'
package-repositories:
- components:
  - main
  formats:
  - deb
  key-id: C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
  key-server: keyserver.ubuntu.com
  suites:
  - focal
  type: apt
  url: http://repo.ros2.org/ubuntu/main
```

### The snapcraft.yaml for ROS 2 Galactic

```yaml
name: ros2-talker-listener
version: '0.1'
summary: ROS2 Talker/Listener Example
description: 'This example launches a ROS2 talker and listener.

  '
grade: devel
confinement: strict
base: core20
parts:
  ros-demos:
    plugin: colcon
    source: https://github.com/ros2/demos.git
    source-branch: galactic
    source-subdir: demo_nodes_cpp
    # build-packages:
    # - make
    # - gcc
    # - g++
    stage-packages:
    - ros-galactic-ros2launch
    build-environment:
    - ROS_VERSION: '2'
    - ROS_DISTRO: galactic
  ros2-galactic-extension:
    build-packages:
    - ros-galactic-ros-core
    override-build: install -D -m 0755 launch ${SNAPCRAFT_PART_INSTALL}/snap/command-chain/ros2-launch
    plugin: nil
    source: $SNAPCRAFT_EXTENSIONS_DIR/ros2
apps:
  ros2-talker-listener:
    command: opt/ros/galactic/bin/ros2 launch demo_nodes_cpp talker_listener.launch.py
    plugs:
    - network
    - network-bind
    command-chain:
    - snap/command-chain/ros2-launch
    environment:
      PYTHONPATH: $SNAP/opt/ros/galactic/lib/python3.8/site-packages:$SNAP/usr/lib/python3/dist-packages:${PYTHONPATH}
      ROS_DISTRO: galactic
      ROS_VERSION: '2'
package-repositories:
- components:
  - main
  formats:
  - deb
  key-id: C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
  key-server: keyserver.ubuntu.com
  suites:
  - focal
  type: apt
  url: http://repo.ros2.org/ubuntu/main
```

## Build the snap

## Test the snap

[ros2-galactic]: https://docs.ros.org/en/galactic/index.html
[ros2-rolling]: https://docs.ros.org/en/rolling/index.html
[foxy-snap]: https://snapcraft.io/blog/how-to-build-a-snap-using-ros-2-foxy