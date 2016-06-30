cmake_minimum_required(VERSION 2.8.3)
project(yumi_description)

find_package(catkin REQUIRED COMPONENTS
  xacro
)

catkin_package()

# 'prefix' argument to urdf
if(NOT YUMI_HARDWARE_INTERFACE)
  set(YUMI_HARDWARE_INTERFACE "EffortJointInterface")
endif()

set(YUMI_XACRO ${CMAKE_CURRENT_SOURCE_DIR}/urdf/yumi.urdf.xacro)
set(YUMI_URDF_DIR ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_SHARE_DESTINATION}/urdf)
set(YUMI_URDF ${YUMI_URDF_DIR}/yumi.urdf)

file(MAKE_DIRECTORY "${YUMI_URDF_DIR}")

# Call out to xacro to get dependencies
execute_process(COMMAND ${CATKIN_ENV} rosrun xacro xacro --deps ${YUMI_XACRO}
  ERROR_VARIABLE _xacro_err_ignore
  OUTPUT_VARIABLE _xacro_deps_result
  OUTPUT_STRIP_TRAILING_WHITESPACE)

separate_arguments(_xacro_deps_result)

add_custom_command(OUTPUT ${YUMI_URDF}
  COMMENT "Generating YuMi URDF"
  COMMAND ${CATKIN_ENV} rosrun xacro xacro
  ARGS -o ${YUMI_URDF} ${YUMI_XACRO} ${_xacro_deps_result} prefix:=${YUMI_HARDWARE_INTERFACE}
  DEPENDS ${YUMI_XACRO} ${_xacro_deps_result})

# add target to actually cause generation
add_custom_target(description_files ALL DEPENDS ${YUMI_URDF})

install(FILES ${YUMI_URDF}
	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/urdf/)

install(
	DIRECTORY gazebo launch meshes
	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
