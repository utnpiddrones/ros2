# Los pasos de instalación están basados en la documentación de PX4
# https://docs.px4.io/master/en/ros/ros2_comm.html 

FROM ros:foxy-ros-base-focal

###############################################################################
#   Dependencias
###############################################################################
RUN apt-get update && apt-get install -y \
    pip \
    curl \
    unzip \
    zip \
    default-jdk \
    git \
    xmlstarlet \
    && rm -rf /var/lib/apt/lists/*

###############################################################################
#   Paquetes de ROS
###############################################################################
RUN apt-get update && apt-get install -y \
    ~nros-${ROS_DISTRO}-rqt* \
    ros-${ROS_DISTRO}-rmw-fastrtps-cpp \
    ros-${ROS_DISTRO}-eigen3-cmake-module \
    ros-${ROS_DISTRO}-gazebo-ros-pkgs \
    python3-colcon-common-extensions \
    && rm -rf /var/lib/apt/lists/*

RUN pip3 install -U \
    empy \
    pyros-genmsg \
    setuptools

###############################################################################
#   Instalación de FAST RTPS (FAST DDS no es necesario al tener ROS2)
###############################################################################
# Source: https://docs.px4.io/master/en/dev_setup/fast-dds-installation.html 

RUN /bin/bash -c "curl -s "https://get.sdkman.io" | bash; \ 
source /root/.sdkman/bin/sdkman-init.sh; \
sdk install gradle 6.3; \
git clone --recursive https://github.com/eProsima/Fast-DDS-Gen.git -b v1.0.4 /root/Fast-RTPS-Gen; \
cd /root/Fast-RTPS-Gen; \
gradle assemble; \
gradle install"


###############################################################################
#   Instalación de los paquetes de PX4 (px4_ros_com y px4_msgs)
###############################################################################
# Source: https://docs.px4.io/main/en/ros/ros2_comm.html 
RUN /bin/bash -c "cd /root; \
git clone https://github.com/PX4/px4_ros_com.git /root/px4_ros_com_ros2/src/px4_ros_com; \
git clone https://github.com/PX4/px4_msgs.git /root/px4_ros_com_ros2/src/px4_msgs"

RUN /bin/bash -c "cd /root/px4_ros_com_ros2; \
source /opt/ros/${ROS_DISTRO}/setup.bash; \
colcon build --cmake-args -DCMAKE_BUILD_TYPE=RELWITHDEBINFO --symlink-install --packages-skip ros1_bridge"

###############################################################################
#   Limpieza
###############################################################################
WORKDIR /root/px4_ros_com_ros2

# Se copia el entrypoint
COPY entrypoint.sh /entrypoint.sh

EXPOSE 2019
EXPOSE 2020

ENTRYPOINT [ "/entrypoint.sh" ]
CMD ["/bin/bash"]
