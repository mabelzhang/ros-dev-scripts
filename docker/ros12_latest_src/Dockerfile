# ROS 1 and 2 latest from source

# 16 Apr 2020

# Built from ../ros2_latest_src
FROM ros2_latest_src:latest


USER root

# ROS 1 Noetic from source, adapted from Melodic instructions
# http://wiki.ros.org/melodic/Installation/Source

# This avoids keyboard interaction when asked for geographic area
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
 && apt-get install -y \
        python3-rosdep \
        python3-rosinstall-generator \
        python3-wstool \
        python3-rosinstall \
        build-essential
#RUN apt-get install -y python-catkin-tools


USER developer
# rosdep init already done in ros2_latest_src
RUN sudo rosdep update


# Resolve dependencies and install ROS 1 Noetic from source
USER root
ENV ROS_WS /opt/ros_ws/noetic
RUN mkdir -p $ROS_WS/src
#RUN apt-get update && apt-get install --only-upgrade python-catkin-pkg
RUN cd $ROS_WS && \
  rosinstall_generator ros_comm --rosdistro noetic --deps --tar > noetic-ros-comm.rosinstall && \
  wstool init -j8 src noetic-ros-comm.rosinstall
RUN rosdep install --from-paths $ROS_WS/src --ignore-src --rosdistro noetic -y
# Download latest version of catkin_make_isolated with python3 in shebang. Else error when run it
RUN cd $ROS_WS/src/catkin/bin/ && \
  wget https://raw.githubusercontent.com/ros/catkin/noetic-devel/bin/catkin_make_isolated -O catkin_make_isolated
RUN cd $ROS_WS && \
  ${ROS_WS}/src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release


USER developer
