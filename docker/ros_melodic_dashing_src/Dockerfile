# ROS 1 Melodic and ROS 2 Dashing from source

# 13 Sep 2019

FROM ubuntu:bionic

# Avoid keyboard interaction
ARG DEBIAN_FRONTEND=noninteractive


#####
# ROS 2 Dashing from source
# Following instructions to install ROS 2 from source
#   https://index.ros.org/doc/ros2/Installation/Dashing/Linux-Development-Setup/

# Set Locale
RUN apt-get update && apt-get install -y locales && \
  locale-gen en_US en_US.UTF-8 && \
  update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && \
  export LANG=en_US.UTF-8

# Add ROS 2 apt repository
# setup keys
RUN apt-get update && apt-get install -y curl gnupg2 lsb-release && \
  curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -
# setup sources.list
RUN sh -c 'echo "deb [arch=amd64,arm64] http://packages.ros.org/ros2/ubuntu `lsb_release -cs` main" > /etc/apt/sources.list.d/ros2-latest.list'


#####
# Create a user, so that can run rosdep update from non-root user
# From subt Docker for gazebo9
#   https://bitbucket.org/osrf/subt/src/gazebo9/docker/subt/Dockerfile

RUN apt-get update \
 && apt-get install -y \
    sudo \
    vim

# Add a user with the same user_id as the user outside the container
# Requires a docker build argument `user_id`
ARG user_id
ENV USERNAME developer
RUN useradd -U --uid ${user_id} -ms /bin/bash $USERNAME \
 && echo "$USERNAME:$USERNAME" | chpasswd \
 && adduser $USERNAME sudo \
 && echo "$USERNAME ALL=NOPASSWD: ALL" >> /etc/sudoers.d/$USERNAME


# Install development tools and ROS tools
RUN apt-get update && apt-get install -y \
  build-essential \
  cmake \
  git \
  python3-colcon-common-extensions \
  python3-pip \
  python-rosdep \
  python3-vcstool \
  wget
# install some pip packages needed for testing
RUN python3 -m pip install -U \
  argcomplete \
  flake8 \
  flake8-blind-except \
  flake8-builtins \
  flake8-class-newline \
  flake8-comprehensions \
  flake8-deprecated \
  flake8-docstrings \
  flake8-import-order \
  flake8-quotes \
  pytest-repeat \
  pytest-rerunfailures \
  pytest \
  pytest-cov \
  pytest-runner \
  setuptools
# install Fast-RTPS dependencies
RUN apt-get install --no-install-recommends -y \
  libasio-dev \
  libtinyxml2-dev

# clone source
ENV ROS2_WS /opt/ros2_ws/dashing
RUN mkdir -p $ROS2_WS/src
RUN cd $ROS2_WS && \
  wget https://raw.githubusercontent.com/ros2/ros2/dashing/ros2.repos && \
  vcs import src < ros2.repos

# Install dependencies using rosdep
# Use non-root for rosdep
USER $USERNAME
# Set env var again after changing user name
ENV ROS2_WS /opt/ros2_ws/dashing
RUN sudo rosdep init && \
  rosdep update && \
  rosdep install --from-paths $ROS2_WS/src --ignore-src --rosdistro dashing -y --skip-keys "console_bridge fastcdr fastrtps libopensplice67 libopensplice69 rti-connext-dds-5.3.1 urdfdom_headers"

USER root
ENV ROS2_WS /opt/ros2_ws/dashing

# build source
RUN cd $ROS2_WS && \
  colcon build \
    --cmake-args \
      -DSECURITY=ON --no-warn-unused-cli \
    --symlink-install

# More DDS implementations
RUN apt-get install libopensplice69

# Custom packages
RUN apt-get install -y \
    libsdl-image1.2-dev \
    libsdl-dev \
    libboost-dev

# setup bashrc
RUN cp /etc/skel/.bashrc ~/


#####
# ROS 1 Melodic from source
# FOllowing instructions to install ROS 1 from source
#   http://wiki.ros.org/melodic/Installation/Source

RUN apt-get update && \
  apt-get install -y python-rosinstall-generator python-wstool python-rosinstall

# Create a catkin workspace
ENV ROS1_WS /opt/ros1_ws/melodic
RUN mkdir -p ${ROS1_WS}/src
# Install Debian
RUN cd ${ROS1_WS} && \
  rosinstall_generator desktop_full --rosdistro melodic --deps --tar > melodic-desktop-full.rosinstall && \
  wstool init -j8 ${ROS1_WS}/src melodic-desktop-full.rosinstall

# Switch to non-root user to run rosdep update
USER $USERNAME
# Set env var again after changing user name
ENV ROS1_WS /opt/ros1_ws/melodic

# Resolve dependencies
RUN rosdep update && \
  rosdep install --from-paths ${ROS1_WS}/src --ignore-src --rosdistro melodic -y

USER root
# Build catkin workspace
# cd into the correct directory first, so that catkin can find relative
#   directory src/
RUN cd ${ROS1_WS} && \
  ${ROS1_WS}/src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space ${ROS1_WS}/install_isolated


# This makes container start up in user instead of root
# Commands below run as the developer user
USER $USERNAME

# When running a container start in the developer's home folder
WORKDIR /home/$USERNAME
