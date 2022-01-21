cmake_minimum_required(VERSION 2.8.3)
project(mujoco_ros)

add_compile_options(-std=c++11 -Ofast)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
  mujoco_ros_msgs
  eigen_conversions
)

find_package(tocabi_lib COMPONENTS)
IF(tocabi_lib_FOUND)
 message(" -- shm mode enabled" )
   ADD_DEFINITIONS(-DCOMPILE_SHAREDMEMORY)
ENDIF()

catkin_package(
DEPENDS   mujoco_ros_msgs
)

if(NOT DEFINED $ENV{MUJOCO_LOCATION})
  set(MUJOCO_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/../mujoco210)
else()
  set(MUJOCO_LOCATION $ENV{MUJOCO_LOCATION})
endif()

message(STATUS ${MUJOCO_LOCATION})

# set(MUJOCO_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/../mujoco210)

set(MUJOCO_INCLUDE_DIR ${MUJOCO_LOCATION}/include)
set(MUJOCO_BIN ${MUJOCO_LOCATION}/bin)


## Specify additional locations of header files
include_directories( 
  include
  ${Boost_INCLUDE_DIR}
  ${catkin_INCLUDE_DIRS}
  ${MUJOCO_INCLUDE_DIR}
  ${EIGEN3_INCLUDE_DIRS}
)

link_directories(
  ${MUJOCO_BIN}
  )

## find libmujoco
file(GLOB LIB_MUJOCO ${MUJOCO_BIN}/libmujoco[0-9][0-9][0-9].so)
file(GLOB UI_SRC "${MUJOCO_INCLUDE_DIR}/uitools.c")

IF(tocabi_lib_FOUND)
include_directories(
  ${tocabi_lib_INCLUDE_DIRS}
)
ENDIF()

add_library(mujoco_ros_lib
    ${UI_SRC}
    src/mjros.cpp
    )

add_executable(mujoco_ros src/main.cpp)

target_link_libraries(mujoco_ros
   mujoco_ros_lib

   ${catkin_LIBRARIES}
   ${LIB_MUJOCO}
   GL
   #GLU 
   #glut
   
   ${MUJOCO_BIN}/libglew.so
   ${MUJOCO_BIN}/libglfw.so.3
 )

add_dependencies(mujoco_ros_lib ${catkin_EXPORTED_TARGETS})
add_dependencies(mujoco_ros ${catkin_EXPORTED_TARGETS})

