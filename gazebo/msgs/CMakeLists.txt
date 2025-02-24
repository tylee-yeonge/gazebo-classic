# Cross-compilation related options
# In a cross-compilation scenario, it is possible that the gazebomsgs_out
# generator compiled for the target machine cannot be used to generate
# the C++ code corresponding to the .proto definition. For this scenario, 
# the following two options can be used as follows. 
# First of all, gazebo is compiled targeting the host machine, and in the
# build targeting the host, the INSTALL_GAZEBOMSGS_OUT_EXECUTABLE option is
# enabled:
# > cmake -DINSTALL_GAZEBOMSGS_OUT_EXECUTABLE:BOOL=ON .. 
# ensuring that the gazebomsgs_out is installed in 
# <host_install_prefix>/bin/gazebomsgs_out . Then, the same version of gazebo
# can be cross-compiled, and in the cross-compilation build the location of the
# host gazebomsgs_out is specified via the GAZEBOMSGS_OUT_EXECUTABLE 
# CMake cache variable: 
# > cmake -DGAZEBOMSGS_OUT_EXECUTABLE=<host_install_prefix>/bin/gazebomsgs_out .. 

option(
  INSTALL_GAZEBOMSGS_OUT_EXECUTABLE
  "Install the gazebomsgs_out executable."
  OFF)
mark_as_advanced(INSTALL_GAZEBOMSGS_OUT_EXECUTABLE)

set(
  GAZEBOMSGS_OUT_EXECUTABLE
  "$<TARGET_FILE:gazebomsgs_out>"
  CACHE STRING
  "gazebomsgs_out executable used in gazebo protobuf msgs generation.")
mark_as_advanced(GAZEBOMSGS_OUT_EXECUTABLE)

include (${gazebo_cmake_dir}/GazeboUtils.cmake)
include_directories(${IGNITION-MATH_INCLUDE_DIRS} ${IGNITION-MSGS_INCLUDE_DIRS})

set (msgs
  altimeter.proto
  any.proto
  atmosphere.proto
  axis.proto
  battery.proto
  boxgeom.proto
  camera_cmd.proto
  camera_lens.proto
  camerasensor.proto
  cessna.proto
  collision.proto
  color.proto
  contact.proto
  contacts.proto
  contactsensor.proto
  cylindergeom.proto
  density.proto
  diagnostics.proto
  distortion.proto
  empty.proto
  factory.proto
  fluid.proto
  fog.proto
  friction.proto
  geometry.proto
  gps.proto
  gps_sensor.proto
  gui.proto
  gui_camera.proto
  gz_string.proto
  gz_string_v.proto
  header.proto
  heightmapgeom.proto
  hydra.proto
  image.proto
  image_stamped.proto
  imagegeom.proto
  images_stamped.proto
  imu.proto
  imu_sensor.proto
  inertial.proto
  int.proto
  joint.proto
  joint_animation.proto
  joint_cmd.proto
  joint_wrench.proto
  joint_wrench_stamped.proto
  joystick.proto
  laserscan.proto
  laserscan_stamped.proto
  light.proto
  link.proto
  link_data.proto
  log_control.proto
  log_playback_control.proto
  log_playback_stats.proto
  log_status.proto
  logical_camera_image.proto
  logical_camera_sensor.proto
  magnetometer.proto
  material.proto
  meshgeom.proto
  model.proto
  model_configuration.proto
  model_v.proto
  packet.proto
  param.proto
  param_v.proto
  performance_metrics.proto
  physics.proto
  pid.proto
  planegeom.proto
  plugin.proto
  pointcloud.proto
  polylinegeom.proto
  pose.proto
  pose_animation.proto
  pose_stamped.proto
  pose_trajectory.proto
  pose_v.proto
  poses_stamped.proto
  projector.proto
  propagation_grid.proto
  propagation_particle.proto
  publish.proto
  publishers.proto
  quaternion.proto
  raysensor.proto
  request.proto
  response.proto
  rest_login.proto
  rest_logout.proto
  rest_post.proto
  rest_response.proto
  road.proto
  scene.proto
  selection.proto
  sensor.proto
  sensor_noise.proto
  server_control.proto
  shadows.proto
  sim_event.proto
  sky.proto
  sonar.proto
  sonar_stamped.proto
  spheregeom.proto
  spherical_coordinates.proto
  subscribe.proto
  surface.proto
  tactile.proto
  test.proto
  time.proto
  topic_info.proto
  track_visual.proto
  twist.proto
  undo_redo.proto
  user_cmd.proto
  user_cmd_stats.proto
  vector2d.proto
  vector3d.proto
  visual.proto
  wind.proto
  wireless_node.proto
  wireless_nodes.proto
  world_control.proto
  world_modify.proto
  world_reset.proto
  world_stats.proto
  wrench.proto
  wrench_stamped.proto
)

set (msgs_tests_sources
  msgs_TEST.cc
  MsgFactory_TEST.cc
)
gz_build_tests(${msgs_tests_sources} EXTRA_LIBS gazebo_msgs)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

add_executable(gazebomsgs_out generator/GazeboGenerator.cc generator/gazebo_generator.cc)
target_link_libraries(gazebomsgs_out ${GZ_PROTOBUF_LIBRARY} ${GZ_PROTOBUF_PROTOC_LIBRARY})
if (UNIX)
  target_link_libraries(gazebomsgs_out pthread)
endif()
# Visual Studio enables c++11 support by default
if (NOT MSVC)
  if(CMAKE_VERSION VERSION_LESS 3.8.2)
    target_compile_options(gazebomsgs_out PRIVATE -std=c++11)
  else()
    target_compile_features(gazebomsgs_out PRIVATE cxx_std_11)
  endif()
endif()

set (sources msgs.cc MsgFactory.cc)
set (headers msgs.hh MsgFactory.hh)

###########################################################
# Append str to a string property of a target.
# target: string: target name.
# property: name of target's property. e.g: COMPILE_FLAGS, or LINK_FLAGS
# str: string: string to be appended to the property
macro(my_append_target_property target property str)
  get_target_property(current_property ${target} ${property})
  if(NOT current_property) # property non-existent or empty
      set_target_properties(${target} PROPERTIES GENERATED TRUE ${property} ${str})
  else()
      set_target_properties(${target} PROPERTIES ${property} "${current_property} ${str}")
  endif()
endmacro(my_append_target_property)

set(PROTO_SRCS)
set(PROTO_HDRS)
foreach(FIL ${msgs})
  get_filename_component(ABS_FIL ${FIL} ABSOLUTE)
  get_filename_component(FIL_WE ${FIL} NAME_WE)

  list(APPEND PROTO_SRCS "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.pb.cc")
  list(APPEND PROTO_HDRS "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.pb.h")

  add_custom_command(
    OUTPUT
      "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.pb.cc"
      "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.pb.h"
    COMMAND  ${PROTOBUF_PROTOC_EXECUTABLE}
    ARGS --plugin=protoc-gen-gazebomsgs=${GAZEBOMSGS_OUT_EXECUTABLE} --cpp_out=dllexport_decl=GZ_MSGS_VISIBLE:${CMAKE_CURRENT_BINARY_DIR} --gazebomsgs_out=${CMAKE_CURRENT_BINARY_DIR} --proto_path=${CMAKE_CURRENT_SOURCE_DIR} ${ABS_FIL}
    DEPENDS ${ABS_FIL} gazebomsgs_out
    COMMENT "Running C++ protocol buffer compiler on ${FIL}"
    VERBATIM )
endforeach()

set_source_files_properties(${PROTO_SRCS} ${PROTO_HDRS} PROPERTIES GENERATED TRUE)
gz_add_library(gazebo_msgs ${PROTO_SRCS} ${sources})
if (UNIX)
  # Need to add default visibility to gazebo_msgs
  get_target_property(current_property gazebo_msgs COMPILE_FLAGS)
  if(NOT current_property) # property non-existent or empty
    set_target_properties(gazebo_msgs PROPERTIES GENERATED TRUE COMPILE_FLAGS "-fvisibility=default")
  else()
    set_target_properties(gazebo_msgs PROPERTIES COMPILE_FLAGS "${current_property} -fvisibility=default")
  endif()
endif()

target_compile_definitions(gazebo_msgs
  PRIVATE BUILDING_DLL_GZ_MSGS
)

target_link_libraries(gazebo_msgs
  gazebo_common
  ${GZ_PROTOBUF_LIBRARY}
  ${IGNITION-MSGS_LIBRARIES}
)

add_dependencies(gazebo_msgs gazebomsgs_out)

set (message_headers "")

foreach (hdr ${PROTO_HDRS})
  string (REPLACE "${CMAKE_CURRENT_BINARY_DIR}/" "" hdr ${hdr})
  set (message_headers "${message_headers}#include \"${hdr}\"\n")
endforeach()
configure_file (${PROJECT_SOURCE_DIR}/gazebo/msgs/MessageTypes.hh.in
                ${CMAKE_CURRENT_BINARY_DIR}/MessageTypes.hh )

gz_install_library(gazebo_msgs)
gz_install_includes("msgs" ${PROTO_HDRS} ${headers} ${CMAKE_CURRENT_BINARY_DIR}/MessageTypes.hh)
gz_install_includes("msgs/proto" ${msgs})

if (INSTALL_GAZEBOMSGS_OUT_EXECUTABLE)
  gz_install_executable(gazebomsgs_out)
endif()
