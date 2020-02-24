# ROS 2 on arm64 architecture

# 20 Feb 2020

FROM ros:dashing-ros-base


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


# Custom packages
RUN apt-get update && apt-get install -y \
    libboost-dev \
    libyaml-cpp-dev \
    vim \
    python3-pytest

# setup bashrc
RUN cp /etc/skel/.bashrc ~/


# This makes container start up in user instead of root
# Commands below run as the developer user
USER $USERNAME
