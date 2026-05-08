cmake_minimum_required(VERSION 3.0.2)
project(ig_lio)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_FLAGS_RELEASE "-O3 ${CMAKE_CXX_FLAGS} -Wall")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")

# ============================================================
# Force using Ubuntu 20.04 legacy TBB instead of /usr/local oneTBB
# ============================================================

list(APPEND CMAKE_IGNORE_PREFIX_PATH "/usr/local")

set(TBB_DIR "/usr/lib/x86_64-linux-gnu/cmake/TBB" CACHE PATH "System legacy TBB path" FORCE)

set(LEGACY_TBB_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/thirdparty/legacy_tbb_include")

set(CMAKE_BUILD_RPATH "/usr/lib/x86_64-linux-gnu")
set(CMAKE_INSTALL_RPATH "/usr/lib/x86_64-linux-gnu")

find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  roslib
  pcl_ros
  livox_ros_driver
)

find_package(Eigen3 REQUIRED)
find_package(PCL 1.8 REQUIRED)

find_package(TBB REQUIRED CONFIG
  PATHS "/usr/lib/x86_64-linux-gnu/cmake/TBB"
  NO_DEFAULT_PATH
)

message(STATUS "Using TBB_DIR: ${TBB_DIR}")
message(STATUS "Using legacy TBB include override: ${LEGACY_TBB_INCLUDE_DIR}")
message(STATUS "Using TBB_VERSION: ${TBB_VERSION}")

include(cmake/glog.cmake)

catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES gicp_lio
#  CATKIN_DEPENDS roscpp rospy std_msgs
#  DEPENDS system_lib
)

add_executable(${PROJECT_NAME}_node 
  src/ig_lio_node.cpp
  src/pointcloud_preprocess.cpp
  src/lio.cpp
  src/voxel_map.cpp
  src/SymmetricEigenSolver.cpp
  src/timer.cpp
  src/faster_voxel_grid.cpp
)

# 这个路径必须放在最前面，用来压住 /usr/local/include/tbb
target_include_directories(${PROJECT_NAME}_node BEFORE PRIVATE
  ${LEGACY_TBB_INCLUDE_DIR}
  ${PROJECT_SOURCE_DIR}/include
  ${PROJECT_SOURCE_DIR}/thirdparty/sophus
)

target_include_directories(${PROJECT_NAME}_node SYSTEM PRIVATE
  ${catkin_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIRS}
)

target_link_libraries(${PROJECT_NAME}_node
  ${catkin_LIBRARIES}
  ${PCL_LIBRARIES}
  glog
  gflags
  TBB::tbb
)

set_target_properties(${PROJECT_NAME}_node PROPERTIES
  BUILD_RPATH "/usr/lib/x86_64-linux-gnu"
  INSTALL_RPATH "/usr/lib/x86_64-linux-gnu"
)

install(TARGETS ${PROJECT_NAME}_node
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
  PATTERN ".svn" EXCLUDE
)

install(DIRECTORY launch/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
  PATTERN ".svn" EXCLUDE
)

install(DIRECTORY config/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/config
  PATTERN ".svn" EXCLUDE
)

install(DIRECTORY rviz/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/rviz
  PATTERN ".svn" EXCLUDE)