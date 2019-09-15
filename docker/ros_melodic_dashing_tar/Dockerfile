# ROS 1 Melodic from Debian, ROS 2 Dashing from tar file release

FROM ubuntu:bionic

# Bypass geographic area prompt
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y locales && \
    locale-gen en_US en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && \
    export LANG=en_US.UTF-8

RUN apt-get update && apt-get install -y curl gnupg2 lsb-release && \
    curl http://repo.ros2.org/repos.key | apt-key add - && \
    sh -c 'echo "deb [arch=amd64,arm64] http://packages.ros.org/ros2/ubuntu `lsb_release -cs` main" > /etc/apt/sources.list.d/ros2-latest.list'


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

# Commands below run as the developer user
USER $USERNAME

# When running a container start in the developer's home folder
WORKDIR /home/$USERNAME

USER root


#####
# ROS 2 Dashing from tarball

RUN apt-get update && apt-get install -y wget && \
    mkdir /home/$USERNAME/Downloads && cd /home/$USERNAME/Downloads && \
    wget https://ci.ros2.org/view/packaging/job/packaging_linux/lastSuccessfulBuild/artifact/ws/ros2-package-linux-x86_64.tar.bz2

RUN mkdir -p /home/$USERNAME/ros2_install && \
    cd /home/$USERNAME/ros2_install && \
    tar xf /home/$USERNAME/Downloads/ros2-package-linux-x86_64.tar.bz2

# Install and init rosdep
RUN apt-get install -y python-rosdep && \
    rosdep init && \
    rosdep update

# Install missing dependencies
RUN rosdep install --from-paths /home/$USERNAME/ros2_install/ros2-linux/share --ignore-src --rosdistro dashing -y --skip-keys "console_bridge fastcdr fastrtps libopensplice67 libopensplice69 osrf_testing_tools_cpp poco_vendor rmw_connext_cpp rosidl_typesupport_connext_c rosidl_typesupport_connext_cpp rti-connext-dds-5.3.1 tinyxml_vendor tinyxml2_vendor urdfdom urdfdom_headers"

# Install python3 libs
RUN apt-get install -y libpython3-dev

# Install additional DDS implementation
RUN apt-get update && sudo apt-get install -q -y libopensplice69


#####
# ROS 1 Melodic from Debian

# Set up sources.list
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

# Set up keys
RUN apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

# Install Melodic
RUN apt-get update \
  && apt-get install -y ros-melodic-desktop-full

# Dependencies for building packages
RUN apt-get install -y python-rosinstall python-rosinstall-generator python-wstool build-essential


# Install colcon for ROS 2 development
RUN apt-get update && apt-get install -y python3-colcon-common-extensions

# This makes container start up in user instead of root
# Commands below run as the developer user
USER $USERNAME
