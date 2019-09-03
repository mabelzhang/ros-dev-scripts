# 27 Aug 2019
# Following instructions to install ROS 2 from source
#   https://index.ros.org/doc/ros2/Installation/Dashing/Linux-Development-Setup/

FROM ubuntu:bionic

# Set Locale
RUN apt update && apt install -y locales && \
  locale-gen en_US en_US.UTF-8 && \
  update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && \
  export LANG=en_US.UTF-8

# Add ROS 2 apt repository
# setup keys
RUN apt update && apt install -y curl gnupg2 lsb-release && \
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
RUN apt update && apt install -y \
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
RUN apt install --no-install-recommends -y \
  libasio-dev \
  libtinyxml2-dev

# clone source
ENV ROS2_WS /opt/ros2_ws
RUN mkdir -p $ROS2_WS/src
RUN cd $ROS2_WS && \
  wget https://raw.githubusercontent.com/ros2/ros2/dashing/ros2.repos && \
  vcs import src < ros2.repos

# Install dependencies using rosdep
# Use non-root for rosdep
USER $USERNAME
# Set env var again after changing user name
ENV ROS2_WS /opt/ros2_ws
# TODO: Fix to use $ROS2_WS
RUN sudo rosdep init && \
  rosdep update && \
  rosdep install --from-paths /opt/ros2_ws/src --ignore-src --rosdistro dashing -y --skip-keys "console_bridge fastcdr fastrtps libopensplice67 libopensplice69 rti-connext-dds-5.3.1 urdfdom_headers"

USER root
ENV ROS2_WS /opt/ros2_ws

# build source
RUN cd $ROS2_WS && \
  colcon build \
    --cmake-args \
      -DSECURITY=ON --no-warn-unused-cli \
    --symlink-install

# More DDS implementations
RUN apt install libopensplice69

# Custom packages
RUN apt-get install -y \
    libsdl-image1.2-dev \
    libsdl-dev \
    libboost-dev

# setup bashrc
RUN cp /etc/skel/.bashrc ~/

# setup entrypoint
#COPY ./ros_entrypoint.sh /
#ENTRYPOINT ["/ros_entrypoint.sh"]
#CMD ["bash"]


# This makes container start up in user instead of root
# Commands below run as the developer user
USER $USERNAME
