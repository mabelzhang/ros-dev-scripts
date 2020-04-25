# Dependencies to build a ROS 2 workspace from source, not including ROS 2,
# (which is to be pulled and built in the container using a directory mounted
# from host machine).

# 24 Apr 2020
# Commands from instructions to install ROS 2 from source
#   https://index.ros.org/doc/ros2/Installation/Foxy/Linux-Development-Setup/#linux-latest

FROM ubuntu:focal

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
# From subt_docker

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

# Commands below run as the developer user
USER $USERNAME

# When running a container start in the developer's home folder
WORKDIR /home/$USERNAME

USER root


# Install development tools and ROS tools
# Avoid keyboard interaction
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y \
  build-essential \
  cmake \
  git \
  python3-colcon-common-extensions \
  python3-flake8 \
  python3-pip \
  python3-pytest-cov \
  python3-rosdep \
  python3-setuptools \
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
  pytest
# install Fast-RTPS dependencies
RUN apt-get install --no-install-recommends -y \
  libasio-dev \
  libtinyxml2-dev
# install CycloneDDS dependencies
#RUN apt-get install --no-install-recommends -y \
#  libcunit1-dev

# argcomplete for ROS 2 command line tools autocomplete
RUN apt-get install -y python3-argcomplete

# Custom packages
RUN apt-get install -y \
    libsdl-image1.2-dev \
    libsdl-dev \
    libboost-dev \
    libyaml-cpp-dev

# Optional: pybind
# Not in Ubuntu 20.04
#RUN apt-get update && apt-get install -y python-pybind11
RUN apt-get update && apt-get install -y python3-argcomplete
RUN apt-get install -y libssl-dev

RUN rosdep init

# setup bashrc
RUN cp /etc/skel/.bashrc ~/


# This makes container start up in user instead of root
# Commands below run as the developer user
USER $USERNAME



# After starting container, run in a directory mounted from host machine, to
# keep the updated workspace on host machine, so that it does not need to be
# updated every time the container restarts:

# clone source
#export ROS2_WS=/home/$USERNAME/ros2_ws/foxy
#mkdir -p ${ROS2_WS}/src
#cd ${ROS2_WS} && \
#  wget https://raw.githubusercontent.com/ros2/ros2/master/ros2.repos && \
#  vcs import src < ros2.repos

# Install dependencies using rosdep
# Dependency for one of the packages
#rosdep update && \
#  sudo apt-get update && \
#  rosdep install --from-paths ${ROS2_WS}/src --ignore-src --rosdistro foxy -y --skip-keys "console_bridge fastcdr fastrtps libopensplice67 libopensplice69 rti-connext-dds-5.3.1 urdfdom_headers"

# build source
# test_rclcpp has compiler error. Temporary skipping
#cd ${ROS2_WS} && \
#  colcon build \
#    --cmake-args \
#      -DSECURITY=ON --no-warn-unused-cli \
#    --symlink-install \
#    --packages-skip test_rclcpp
