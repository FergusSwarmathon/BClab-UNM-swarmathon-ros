cmake_minimum_required(VERSION 2.8.3)
project(rqt_rover_gui)

set(CMAKE_BUILD_TYPE Debug)

# QT5 Support. QT5 is needed for compatability with recent boost libraries.
# Tell CMake to run moc when necessary:
set(CMAKE_AUTOMOC ON)

# As moc files are generated in the binary dir, tell CMake
# to always look for includes there:
set(CMAKE_INCLUDE_CURRENT_DIR ON)

# Widgets finds its own dependencies (QtGui and QtCore).
find_package(Qt5Widgets REQUIRED)

# The Qt5Widgets_INCLUDES also includes the include directories for
# dependencies QtCore and QtGui. Also include the ROS headers
include_directories(${Qt5Widgets_INCLUDES} ${catkin_INCLUDE_DIRS})

# We need add -DQT_WIDGETS_LIB when using QtWidgets in Qt 5.
add_definitions(${Qt5Widgets_DEFINITIONS})

# Executables fail to build with Qt 5 in the default configuration
# without -fPIE. We add that here.
set(CMAKE_CXX_FLAGS "${Qt5Widgets_EXECUTABLE_COMPILE_FLAGS}")

#Get version info from git and pass it into compiled code
execute_process(COMMAND git describe --tags --abbrev=0
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
    OUTPUT_VARIABLE GIT_VERSION)
string(STRIP "${GIT_VERSION}" GIT_VERSION)

execute_process(COMMAND date "+%Y-%m-%d-%H:%M"
    OUTPUT_VARIABLE BUILD_TIME)
string(STRIP ${BUILD_TIME} BUILD_TIME)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/Version.cpp.in ${CMAKE_CURRENT_BINARY_DIR}/Version.cpp)
set(version_file "${CMAKE_CURRENT_BINARY_DIR}/Version.cpp")

set(CMAKE_CXX_FLAGS "-std=c++0x ${CMAKE_CXX_FLAGS}")

find_package(catkin REQUIRED COMPONENTS 
  rqt_gui
  rqt_gui_cpp
  cv_bridge
  image_transport
  shared_messages
  geometry_msgs
)

find_package(gazebo REQUIRED)
include_directories(${GAZEBO_INCLUDE_DIRS})
link_directories(${GAZEBO_LIBRARY_DIRS})
#list(APPEND CMAKE_CXX_FLAGS "${GAZEBO_CXX_FLAGS}")

catkin_package(
  CATKIN_DEPENDS rqt_gui rqt_gui_cpp cv_bridge image_transport shared_messages geometry_msgs
)

SET(rover_gui_plugin_RESOURCES resources/resources.qrc)

qt5_wrap_cpp(
  rover_gui_plugin_MOCS
  src/rover_gui_plugin.h
  src/CameraFrame.h
  src/MapFrame.h
  src/USFrame.h
  src/GPSFrame.h
  src/IMUFrame.h
  src/JoystickGripperInterface.h
  #src/IMUWidget.h
)

qt5_wrap_ui(
  rover_gui_plugin_UIS_H
  src/rover_gui_plugin.ui
)

include_directories(
  include
  src
  ${CMAKE_CURRENT_BINARY_DIR}
)

link_directories(
  lib
)

add_library(
  rqt_rover_gui
  ${version_file}
  src/GazeboSimManager.cpp
  src/JoystickGripperInterface.cpp
  src/rover_gui_plugin.cpp
  src/CameraFrame.cpp
  src/MapFrame.cpp
  src/USFrame.cpp
  src/GPSFrame.cpp
  #src/IMUWidget.cpp
  src/IMUFrame.cpp
  src/BWTabWidget.cpp
  ${rover_gui_plugin_RESOURCES}
  ${rover_gui_plugin_MOCS}
  ${rover_gui_plugin_UIS_H}
  ${OpenCV_LIBS}
)

add_dependencies(rqt_rover_gui ${catkin_EXPORTED_TARGETS})

target_link_libraries(
  rqt_rover_gui
  libapriltag.a	
  ${GAZEBO_libraries}
  ${catkin_LIBRARIES}
  #The following includes the QtCore and QtGui
  ${Qt5Widgets_LIBRARIES} 
)

catkin_python_setup()

set(CMAKE_BUILD_TYPE Debug)
