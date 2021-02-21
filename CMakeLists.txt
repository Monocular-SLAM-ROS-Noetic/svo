cmake_minimum_required(VERSION 3.0.2)
project(svo_analysis)

find_package(catkin REQUIRED COMPONENTS rospy)
include_directories(${catkin_INCLUDE_DIRS})

catkin_python_setup()
#catkin_package(CATKIN_DEPENDS sensor_msgs)
