# colcon-bundle

This code is in active development and **should not** be considered stable.

This package is a plugin to [colcon-core](https://github.com/colcon/colcon-core.git). It provides functionality to bundle a built
workspace. The bundle is essentially a portable environment which can be moved to a different linux system and executed
as if the contents of the bundle was installed locally. A bundle should work on any Linux system with a kernel version
equal to or later than the kernel version that the bundle was built on. 

Currently, ROS Kinetic is supported by installing the `colcon-ros-bundle` package.

# Usage

To use `colcon-bundle` after building your workspace execute:

`colcon bundle`

This will parse your dependencies, download apt and pip dependencies, install the dependencies into the bundle, and
then install your built workspace into the bundle. The final output is located at `bundle/output.tar.gz`


## Bundle Execution

The bundle file specification is located [here](BUNDLE_FORMAT.md). In order to execute inside the bundle context
follow the following steps:

1. Extract the main archive.
1. Extract bundle.tar into your desired directory.
1. Set the environment variable BUNDLE_CURRENT_PREFIX to the location you extracted bundle.tar to.
1. Source `setup.sh` located in the folder you extracted your bundle in.
1. Everything in your bundle is now in PATH and LD_LIBRARY_PATH so that you can use it as if it were installed on
the system.


# Development

To setup my workspace I generally pull down:

* `colcon-core` (https://github.com/colcon/colcon-core.git)
* `colcon-ros` (https://github.com/colcon/colcon-ros.git)
* `colcon-bundle` (ssh://git.amazon.com/pkg/Colcon-bundle)
* `colcon-ros-bundle` (ssh://git.amazon.com/pkg/Colcon-ros-bundle)

There are many more colcon packages, it can be useful to pull them down to look at how different extensions and other
functionality is implemented.

## Testing

To run tests execute `pytest` in the root directory. Install dependencies using `pip3 install -r requirements_devel.txt`.
You might need to `apt-get install enchant` to install the spellchecker.

To view stdout from a test while running `pytest` use the `-s` flag.

## Running on OSX

I run these packages inside of docker containers since I'm running OSX and we are supporting Ubuntu to start.

Build Container: `docker run -it -v $(pwd):/workspace ros:kinetic-ros-base /bin/bash`

Run Container: `docker run -it -v $(pwd):/workspace ubuntu:xenial /bin/bash`

I generally `cd` into my workspace which has the package folders and then start the container. This docker command
 mounts `pwd` to`/workspace`. Once in the container I `cd /workspace` and then execute:

1. `apt-get update`
1. `apt-get install -y python3-pip python3-apt`
1. `pip3 install --upgrade pip`
1. `export PATH=/usr/local/bin/pip3:$PATH`
1. `/usr/local/bin/pip3 install --editable ./colcon-bundle`
1. `/usr/local/bin/pip3 install --editable ./colcon-ros-bundle`

In `colcon-ros-bundle/test/assets/catkin_ws` there is a valid ros workspace. To build and bundle go inside of the
`catkin_ws` directory and execute the following:

1. `rosdep install --from-paths src --ignore-src -r -y`
1. `colcon build`
With ROS:
    `colcon bundle --bundle-base=bundle_with_ros --include-ros-base`
Without ROS:
    `colcon bundle --bundle-base=bundle_without_ros`  

I use `--base-paths` because I haven't updated the bundle to include a COLCON_IGNORE yet. You can change the name
of the `build` folder with `--build-base=` and the `install` folder with `--install-base=`.

## Running the bundle

To run the bundle you should start up a Xenial docker container with the bundle mounted. You will need
to make some changes to the bundle if you are including ros-base.

To execute inside the bundle you need to set BUNDLE_CURRENT_PREFIX equal to the location of your extracted bundle
folder. Then run `run_command.sh` within the bundle. This is in flux, and may be changed in the future.

### Package Blacklist

When we create the bundle we choose not to include certain packages that are included by default in most
Linuxd distributions. To create this blacklist for Ubuntu I ran the following on a ubuntu:xenial container.

`apt list --installed | sed 's/^\(.*\)\/.*$/\1/'` on a base image.

# Packaging

To create a tarball of this python package run: `python setup.py sdist`

This will create a tarball in the `dist/` directory.