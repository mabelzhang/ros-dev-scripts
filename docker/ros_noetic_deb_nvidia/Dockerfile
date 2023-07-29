# Install ROS 1 Noetic binary packages, with NVIDIA support for running GUI.
# Adapted from official installation instructions
# http://wiki.ros.org/noetic/Installation/Ubuntu

# 29 Jul 2023

# Ubuntu 22.04 with nvidia-docker2 beta opengl support
# From https://github.com/mabelzhang/ign-dev-scripts/tree/master/docker/nvidia_opengl_ubuntu22
FROM nvidia_opengl_ubuntu20

# This avoids keyboard interaction when asked for geographic area
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
 && apt-get install -y \
  curl \
  gnupg2 \
  lsb-release \
 && apt-get clean

# Set up sources.list
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

# Set up keys
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -

# Install ROS
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
  ros-noetic-desktop-full \
 && apt-get clean

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
