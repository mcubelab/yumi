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

set(YUMI_URDF_DIR ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_SHARE_DESTINATION}/urdf)
set(YUMI_URDF_XACRO ${CMAKE_CURRENT_SOURCE_DIR}/urdf/yumi.urdf.xacro)
set(YUMI_URDF ${YUMI_URDF_DIR}/yumi.urdf)
set(YUMI_SRDF_XACRO ${CMAKE_CURRENT_SOURCE_DIR}/urdf/yumi.srdf.xacro)
set(YUMI_SRDF ${YUMI_URDF_DIR}/yumi.srdf)

file(MAKE_DIRECTORY "${YUMI_URDF_DIR}")

# Call out to xacro to get dependencies
macro(build_xacro infile outfile argstring)
    execute_process(COMMAND ${CATKIN_ENV} rosrun xacro xacro --deps ${infile}
    ERROR_VARIABLE _xacro_err_ignore
    OUTPUT_VARIABLE _xacro_deps_result
    OUTPUT_STRIP_TRAILING_WHITESPACE)

    separate_arguments(_xacro_deps_result)

    add_custom_command(OUTPUT ${outfile}
    COMMENT "Generating ${outfile}"
    COMMAND ${CATKIN_ENV} rosrun xacro xacro
    ARGS -o ${outfile} ${infile} ${_xacro_deps_result} ${argstring}
    DEPENDS ${infile} ${_xacro_deps_result})
endmacro(build_xacro)

build_xacro(${YUMI_URDF_XACRO} ${YUMI_URDF} "prefix:=${YUMI_HARDWARE_INTERFACE}")
build_xacro(${YUMI_SRDF_XACRO} ${YUMI_SRDF} "")

# add target to actually cause generation
add_custom_target(description_files ALL DEPENDS
  ${YUMI_URDF}
  ${YUMI_SRDF})

install(FILES
  ${YUMI_URDF}
  ${YUMI_SRDF}
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/urdf/)

install(
	DIRECTORY gazebo launch meshes
	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
