# Build & Run the Camera Streaming Daemon

This topic explains how to setup and run the *Camera Streaming Daemon* (CSD) on a Linux computer.

The build system follows a typical *configure > build > install cycle*. The [configuration](#configure) step allows you enable/disable specific features in the build, thereby choosing the combination of features that best-fit your requirements.

> **Tip** We recommend you read this topic as it fully explains the build process. The *Quickstart* topics (see sidebar) provide additional instructions for setting up common configurations.

<span id="dependencies"></span>
## Prerequisites

The CSD can be configured to enable/disable specific functionality at compile-time. As a result, some dependencies are only required in order to use specific features. 

The following packages are needed to compile CSD with core functionality (RTSP streaming and MAVLink support):

- Autoconf and libtool (for build configuration)
- GCC/G++ compiler 4.9 or newer
- C and C++ standard libraries
- [GLib](https://wiki.gnome.org/Projects/GLib) 2.42 or newer 
- [GStreamer](https://gstreamer.freedesktop.org/) 1.4 or newer 
- [GStreamer RTSP Server](https://gstreamer.freedesktop.org/modules/gst-rtsp-server.html) 1.4 or newer 
- Python 2

Optional packages include:
- [Avahi](https://github.com/lathiat/avahi) 0.6 to enable publishing what RTSP streams are available.
- [RealSense](https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md#installing-the-packages) to support the Intel RealSense 3D Camera.
- [Gazebo](http://gazebosim.org/) in order to enable a simulated camera within the Gazebo environment.


## Installing Prerequisites on Ubuntu

The following sections show how to install the required packages on Ubuntu 16.04 LTS.

### Core Dependencies {#core_deps}

The core dependencies are required to build CSD with MAVLink and RTSP video streaming support.
```sh
sudo apt-get update -y
sudo apt-get install git autoconf libtool python-pip -y
sudo apt-get install gstreamer-1.0 \
    libgstreamer-plugins-base1.0-dev \
    libgstrtspserver-1.0-dev -y
# Required python packages
sudo pip2 -q install -U future
```

> **Note** GCC 5.4 and the C/C++ standard libraries are installed by default.


<span id="avahi_deps"></span>
### Avahi

Run the following command to get the [Avahi](https://github.com/lathiat/avahi) dependencies:
```sh
sudo apt-get install libavahi-client-dev libavahi-core-dev libavahi-glib-dev -y
```

<span id="realsense_deps"></span>
### RealSense 3D Camera

Run the following command to get the [RealSense SDK 1](https://software.intel.com/sites/products/realsense/intro/getting_started.html) libraries:
```
echo 'deb "http://realsense-alm-public.s3.amazonaws.com/apt-repo" xenial main' | sudo tee /etc/apt/sources.list.d/realsense-latest.list
sudo apt-key adv --keyserver keys.gnupg.net --recv-key D6FB2970 
sudo apt update -y
sudo apt-get install librealsense-dev -y
```

<!-- What are runtime dependencies? https://github.com/intel/camera-streaming-daemon/issues/124 -->

> **Note** The RealSense 2 SDK is not supported. 

<span></span>
> **Note** CSD only has access to this camera when it is **not being used** for optical flow or VIO)

<span id="gazebo_deps"></span>
### Gazebo

The easiest way to set up Gazebo and the PX4 simulator to use the *PX4 Developer Guide* scripts: [Development Environment on Linux > jMAVSim/Gazebo Simulation](https://dev.px4.io/en/setup/dev_env_linux.html#jmavsimgazebo-simulation).


### Get the Source Code

Clone the [camera-streaming-daemon](https://github.com/intel/camera-streaming-daemon) repo (or your fork):
```sh
git clone https://github.com/intel/camera-streaming-daemon.git
cd camera-streaming-daemon
git submodule update --init --recursive
```

This fetches all the sources for the project (including the [MAVLink C library](https://mavlink.io/en/) submodule, which is generated by the build system during compilation).

> **Note** Alternatively you can do this in one line:
  ```
  git clone https://github.com/intel/camera-streaming-daemon.git --recursive
  ```

### Serve Camera Definition Files

Before running CSD with MAVLink enabled *on Ubuntu*, you should start serving the sample [Camera Definition Files](../guide/camera_definition_file.md):
1. Open a new terminal to [/samples/def](https://github.com/intel/camera-streaming-daemon/tree/master/samples/def)
1. Enter the following command to start the server on the default port (8000):
   ```
   python -m SimpleHTTPServer
   ```

> **Tip** The [Camera Definition Files](../guide/camera_definition_file.md) is only needed if you are running CSD with MAVLink enabled. 

<span></span>
> **Note** Here we chose to serve the the file from Ubuntu. In fact, the file can be hosted anywhere that is accessible to clients (e.g. QGroundControl, Dronecode SDK), provided you update the [CSD Configuration File](../guide/configuration_file.md) with its URI. For more information see [Camera Definition Files](../guide/camera_definition_file.md).


## Configure

Configuration allows you to specify the features that will be included when CSD is compiled (this step need only be done once).

The full configuration syntax is given below:
```sh
./autogen.sh && ./configure [--enable-mavlink] [--enable-aero] [--enable-avahi] [--enable-gazebo]
```

The optional configuration options enable specific functionality at compile-time:
* `--enable-aero`: Enables aero-specific bottom bottom facing VGA camera (OV7251 sensor for optical flow/VIO).
* `--enable-realsense`: Enables Intel RealSense 3D Camera.
* `--enable-avahi`: Enables Avahi to advertise the list of available RTSP streams.
* `--enable-mavlink`: Enables MAVLink [Camera Protocol](https://mavlink.io/en/protocol/camera.html) support.
* `--enable-gazebo`: Enables Gazebo camera.

In addition, CSD generates the file **csd.service** by default (see [Auto-start CSD](../guide/autostart.md)). File generation can be disabled/configured using:
* `--disable-systemd`: Disable *systemd* support (i.e. on systems where *systemd* is not present).
* `--with-systemdsystemunitdir <path>`: Set the *systemd* system directory to `<path>` (Default is taken from **pkg-config**).

The configuration step will fail gracefully if any [dependency](#dependencies) required by the specified configuration is not available.
At the end of the configuration process the system will output a report showing what additional features are enabled: 
```
RealSense support:   yes
MAVLink support:     yes
AVAHI support:       no
Intel Aero support:  no
Gazebo support:      no
```


## Build

After configuration, build the CSD using *make*:
```
make
```

The *csd* executable will be created in the root of your CSD source tree (along with the Intel Aero CSD startup file: **csd.service**).

## Run{#run_csd}

The line below shows how to start CSD, specifying a [CSD Configuration File](../guide/configuration_file.md) (in this case the Ubuntu **.conf** file in the source tree):
```
./csd -c samples/config/ubuntu.conf
```

> **Tip** The [samples/config](https://github.com/intel/camera-streaming-daemon/tree/master/samples/config) directory contains sample [configuration files](../guide/configuration_file.md) that you can use to set up CSD for use on Ubuntu, Aero and other platforms. To use a sample file, copy it to **/etc/csd/main.conf**, specify it in the `CSD_CONF_FILE` environment variable, or set the `-c` switch when starting CSD.

Other command line options can be displayed using the `-h` flag:
```sh
$ ./csd -h
csd [OPTIONS...]

  -c --conf-file                   .conf file with configurations for camera-streaming-daemon.
  -d --conf-dir <dir>              Directory where to look for .conf files overriding
                                   default conf file.
  -g --debug-log-level <level>     Set debug log level. Levels are
                                   <error|warning|notice|info|debug>
  -v --verbose                     Verbose. Same as --debug-log-level=debug
  -h --help                        Print this message
```

> **Note** CSD can also be started automatically on boot. This is discussed in [Quickstart — Intel Aero](../getting_started/quick_start_intel_aero.md) and [Autostart CSD](../guide/autostart.md).


## Sanity Tests

After installing CSD, use the [Sanity Tests](../test/sanity_tests.md) to verify that CSD is working correctly.