# Install ROS 1 Noetic from source, with NVIDIA support for running GUI such as
#   rqt.
# Adapted from Noetic instructions for installation from source
# http://wiki.ros.org/noetic/Installation/Source

# 15 Oct 2020

# Ubuntu 18.04 with nvidia-docker2 beta opengl support
FROM nvidia/opengl:1.0-glvnd-devel-ubuntu20.04

# This avoids keyboard interaction when asked for geographic area
ARG DEBIAN_FRONTEND=noninteractive

# Set up sources.list and keys. Needed to find python3-vcstool below
# This needs to be done in the very beginning, otherwise rosdep install will
# have problems with python3-catkin-pkg-modules, python3-rospkg-modules, etc.
RUN apt-get update && apt-get install -y curl gnupg2 lsb-release && \
  sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

RUN apt-get update \
 && apt-get install -y \
        build-essential \
        cmake \
        cppcheck \
        gdb \
        git \
        libusb-dev \
        python3-pip \
        python3-rosdep \
        python3-rosinstall-generator \
        python3-vcstool \
        sudo \
        vim \
        wget \
 && apt-get clean

# For some reason this file already exists
RUN rosdep init


# Add a user with the same user_id as the user outside the container
# Requires a docker build argument `user_id`
ARG user_id
ENV USERNAME developer
RUN useradd -U --uid ${user_id} -ms /bin/bash $USERNAME \
 && echo "$USERNAME:$USERNAME" | chpasswd \
 && adduser $USERNAME sudo \
 && echo "$USERNAME ALL=NOPASSWD: ALL" >> /etc/sudoers.d/$USERNAME

# Commands below run as the developer user
USER $USERNAME

# When running a container, start in the non-root user's home folder
WORKDIR /home/$USERNAME

RUN sudo rosdep update


USER root

# Create catkin workspace
ENV ROS_WS /opt/ros_ws/noetic
RUN mkdir -p ${ROS_WS}/src
RUN cd ${ROS_WS} && \
  rosinstall_generator desktop --rosdistro noetic --deps --tar > noetic-desktop.rosinstall && \
  vcs import --input noetic-desktop.rosinstall ${ROS_WS}/src

# Resolve dependencies
RUN rosdep install --from-paths ${ROS_WS}/src --ignore-packages-from-source --rosdistro noetic -y

# Build the catkin workspace
RUN cd $ROS_WS && \
  ${ROS_WS}/src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release


# When running a container, start with the non-root user
USER $USERNAME
