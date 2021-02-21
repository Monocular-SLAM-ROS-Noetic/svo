project(svo_ros)
cmake_minimum_required(VERSION 3.0.2)

#find_package(catkin_simple REQUIRED)
#catkin_simple()

SET(CMAKE_BUILD_TYPE Release) # Release, RelWithDebInfo
SET(CMAKE_VERBOSE_MAKEFILE OFF)
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/CMakeModules/")

#############################################################################
# Set build flags, set ARM_ARCHITECTURE environment variable on Odroid
SET(Sophus_LIBRARIES libSophus.so)
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -Wall -Werror -D_LINUX -D_REENTRANT -Wno-unused-variable -Wno-unused-but-set-variable -Wno-unknown-pragmas")

IF(DEFINED ENV{ARM_ARCHITECTURE})
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfpu=neon -march=armv7-a")
  ADD_DEFINITIONS(-DHAVE_FAST_NEON)
ELSE()
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mmmx -msse -msse -msse2 -msse3 -mssse3 -mno-avx")
ENDIF()
IF(CMAKE_COMPILER_IS_GNUCC)
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
ELSE()
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
ENDIF()
SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O3 -fsee -fomit-frame-pointer -fno-signed-zeros -fno-math-errno -funroll-loops -ffast-math -fno-finite-math-only")

############################################################################

#Incremental change for the new change
# Add catkin and required ROS packages
FIND_PACKAGE(catkin REQUIRED COMPONENTS 
  roscpp
  cmake_modules
  nav_msgs
  std_msgs
  visualization_msgs
  sensor_msgs
  image_transport
  cv_bridge
  tf
  svo
  svo_msgs
  vikit_common
  vikit_ros
)


# Add plain cmake packages 
FIND_PACKAGE(OpenCV REQUIRED)
FIND_PACKAGE(Eigen REQUIRED)
FIND_PACKAGE(Sophus REQUIRED)
FIND_PACKAGE(Boost REQUIRED COMPONENTS thread)

# Describe catkin Project
catkin_package(
  DEPENDS Eigen OpenCV Sophus Boost
  CATKIN_DEPENDS roscpp nav_msgs std_msgs visualization_msgs sensor_msgs image_transport cv_bridge tf svo svo_msgs vikit_common vikit_ros
  INCLUDE_DIRS include
  LIBRARIES svo_visualizer
)

# Include dirs
INCLUDE_DIRECTORIES(
  include
  ${Eigen_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${Sophus_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
)

# Set link libraries
LIST(APPEND LINK_LIBS 
  ${OpenCV_LIBS}
  ${Sophus_LIBRARIES}
  ${Boost_LIBRARIES}
  ${catkin_LIBRARIES}
  /usr/local/lib/libSophus.so
)

# Create visualizer library
ADD_LIBRARY(svo_visualizer src/visualizer.cpp)
TARGET_LINK_LIBRARIES(svo_visualizer ${LINK_LIBS} /usr/local/lib/libSophus.so)
                                 
# Create Executables                      
ADD_EXECUTABLE(vo src/vo_node.cpp)
TARGET_LINK_LIBRARIES(vo svo_visualizer)
ADD_EXECUTABLE(benchmark src/benchmark_node.cpp)
TARGET_LINK_LIBRARIES(benchmark svo_visualizer)

#############################################################################
# Build library

set(HEADERS
  include/svo_ros/svo_interface.h
  include/svo_ros/svo_factory.h
  include/svo_ros/visualizer.h
  include/svo_ros/svo_nodelet.h
)

set(SOURCES
  src/svo_interface.cpp
  src/svo_factory.cpp
  src/svo_node_base.cpp
  src/visualizer.cpp
)

cs_add_library(${PROJECT_NAME} ${SOURCES} ${HEADERS})

#############################################################################
# Build executables
cs_add_executable(svo_node src/svo_node.cpp)
target_link_libraries(svo_node ${PROJECT_NAME})

cs_add_library(svo_nodelet src/svo_nodelet.cpp)
target_link_libraries(svo_nodelet ${PROJECT_NAME})

#############################################################################
# Install
cs_install()
install(FILES svo_nodelet.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
cs_export()
