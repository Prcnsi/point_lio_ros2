cmake_minimum_required(VERSION 3.14)  # Recommended for ROS 2
# define the project name
project(point_lio_ros2)


# Use C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fexceptions")

# Compiler settings for performance and threading
add_definitions(-DROOT_DIR=\"${CMAKE_CURRENT_SOURCE_DIR}/\")
add_compile_options(-O3 -pthread -fexceptions)

# Processor count and definitions
message(STATUS "Current CPU architecture: ${CMAKE_SYSTEM_PROCESSOR}")
if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86)|(X86)|(amd64)|(AMD64)" )
  include(ProcessorCount)
  ProcessorCount(N)
  message(STATUS "Processor number:  ${N}")
  if(N GREATER 5)
    add_definitions(-DMP_EN)
    add_definitions(-DMP_PROC_NUM=4)
    message(STATUS "core for MP:  3")
  elseif(N GREATER 3)
    math(EXPR PROC_NUM "${N} - 2")
    add_definitions(-DMP_EN)
    add_definitions(-DMP_PROC_NUM="${PROC_NUM}")
    message(STATUS "core for MP:  ${PROC_NUM}")
  else()
    add_definitions(-DMP_PROC_NUM=1)
  endif()
else()
  add_definitions(-DMP_PROC_NUM=1)
endif()

# Find packages
find_package(ament_cmake REQUIRED)
find_package(ament_index_cpp REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclpy REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(PCL REQUIRED COMPONENTS common io)  # Core PCL components
find_package(livox_ros_driver2 REQUIRED)

# OpenMP
find_package(OpenMP QUIET)
if(OPENMP_FOUND)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
endif()

# Python & matplotlibcpp
find_package(PythonLibs REQUIRED)
find_path(MATPLOTLIB_CPP_INCLUDE_DIRS "matplotlibcpp.h")

# Include directories
include_directories(
  include
  ${EIGEN3_INCLUDE_DIR}
  ${PYTHON_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
)

# Declare executable
add_executable(pointlio_mapping
  src/laserMapping.cpp
  include/ikd-Tree/ikd_Tree.cpp
  src/parameters.cpp
  src/preprocess.cpp
  src/Estimator.cpp
)

# Link dependencies
ament_target_dependencies(pointlio_mapping
  rclcpp
  rclpy
  geometry_msgs
  nav_msgs
  sensor_msgs
  pcl_conversions
  tf2_ros
  visualization_msgs
  livox_ros_driver2  
)

target_link_libraries(pointlio_mapping
  ${PYTHON_LIBRARIES}
  ${PCL_LIBRARIES}
)

target_include_directories(pointlio_mapping PRIVATE
  ${PYTHON_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
)

# Install
install(TARGETS
  pointlio_mapping
  DESTINATION lib/${PROJECT_NAME}
)

install(
  DIRECTORY config launch rviz_cfg
  DESTINATION share/${PROJECT_NAME}
)

# Export dependencies
ament_export_dependencies(
  rclcpp
  rclpy
  geometry_msgs
  nav_msgs
  sensor_msgs
  pcl_conversions
  tf2_ros
  visualization_msgs
  Eigen3
  PCL
)

ament_package()
