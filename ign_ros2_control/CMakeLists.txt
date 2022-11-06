cmake_minimum_required(VERSION 3.5)
project(ign_ros2_control)

# Default to C11
if(NOT CMAKE_C_STANDARD)
    set(CMAKE_C_STANDARD 11)
endif()
# Default to C++17
if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 17)
endif()

# Compiler options
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Find dependencies
find_package(ament_cmake REQUIRED)
find_package(ament_index_cpp REQUIRED)
find_package(controller_manager REQUIRED)
find_package(hardware_interface REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rclcpp REQUIRED)
find_package(yaml_cpp_vendor REQUIRED)

if("$ENV{IGNITION_VERSION}" STREQUAL "citadel")
  find_package(ignition-gazebo3 REQUIRED)
  set(IGN_GAZEBO_VER ${ignition-gazebo3_VERSION_MAJOR})
  message(STATUS "Compiling against Ignition Citadel")

elseif("$ENV{IGNITION_VERSION}" STREQUAL "edifice")
  find_package(ignition-gazebo5 REQUIRED)
  set(IGN_GAZEBO_VER ${ignition-gazebo5_VERSION_MAJOR})
  message(STATUS "Compiling against Ignition Edifice")

elseif("$ENV{IGNITION_VERSION}" STREQUAL "fortress")
  find_package(ignition-gazebo6 REQUIRED)
  set(IGN_GAZEBO_VER ${ignition-gazebo6_VERSION_MAJOR})
  message(STATUS "Compiling against Ignition Fortress")

elseif("$ENV{IGNITION_VERSION}" STREQUAL "garden")
  find_package(gz-sim7 REQUIRED)
  set(IGN_GAZEBO_VER ${gz-sim7_VERSION_MAJOR})
  message(STATUS "Compiling against Ignition Garden")

else()
  find_package(ignition-gazebo6 REQUIRED)
  set(IGN_GAZEBO_VER ${ignition-gazebo6_VERSION_MAJOR})
  message(STATUS "Compiling against Ignition Fortress")
endif()

find_package(gz-plugin2 REQUIRED)
set(IGN_PLUGIN_VER ${gz-plugin2_VERSION_MAJOR})

include_directories(include)

add_library(${PROJECT_NAME}-system SHARED
  src/ign_ros2_control_plugin.cpp
)

target_link_libraries(${PROJECT_NAME}-system
  gz-sim${IGN_GAZEBO_VER}::core
  gz-plugin${IGN_PLUGIN_VER}::register
)
ament_target_dependencies(${PROJECT_NAME}-system
  ament_index_cpp
  controller_manager
  hardware_interface
  pluginlib
  rclcpp
  yaml_cpp_vendor
  rclcpp_lifecycle
)

#########

add_library(ign_hardware_plugins SHARED
  src/ign_system.cpp
)
ament_target_dependencies(ign_hardware_plugins
  rclcpp_lifecycle
  hardware_interface
  rclcpp
)
target_link_libraries(ign_hardware_plugins
  gz-sim${IGN_GAZEBO_VER}::core
)

## Install
install(TARGETS
  ign_hardware_plugins
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

# Testing and linting
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME} ign_hardware_plugins)

# Install directories
install(TARGETS ${PROJECT_NAME}-system
  DESTINATION lib
)

pluginlib_export_plugin_description_file(ign_ros2_control ign_hardware_plugins.xml)

# Setup the project
ament_package()
