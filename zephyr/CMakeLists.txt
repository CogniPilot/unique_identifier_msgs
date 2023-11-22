# SPDX-License-Identifier: Apache-2.0
if (CONFIG_UNIQUE_IDENTIFIER_MSGS)

zephyr_library_named(unique_identifier_msgs)

target_compile_definitions(unique_identifier_msgs PUBLIC
  ROS_PACKAGE_NAME=rolling
  )

target_compile_options(unique_identifier_msgs PRIVATE -Wno-unused-variable -Wno-implicit-function-declaration -Wno-int-conversion)

zephyr_include_directories(
  ${ZEPHYR_CURRENT_MODULE_DIR}/rclc/include
  ${ZEPHYR_CURRENT_MODULE_DIR}/../rosidl/rosidl_runtime_c/include
  ${ZEPHYR_CURRENT_MODULE_DIR}/../rcutils/include
  )

zephyr_library_include_directories(
  ${ZEPHYR_CURRENT_MODULE_DIR}/rclc/include
  ${ZEPHYR_CURRENT_MODULE_DIR}/../rosidl/rosidl_runtime_c/include
  ${ZEPHYR_CURRENT_MODULE_DIR}/../rcutils/include
  )

file(GLOB_RECURSE Sources
  "${ZEPHYR_CURRENT_MODULE_DIR}/unique_identifier_msgs/src/*.c"
)

#message(STATUS "Sources: ${Sources}")

function(add_package package sources)
  file(GLOB_RECURSE msgs
    RELATIVE ${ZEPHYR_CURRENT_MODULE_DIR}
    "${ZEPHYR_CURRENT_MODULE_DIR}/msg/*.msg"
  )
  message("msg: ${msgs}")
  foreach(msg ${msgs})
    get_filename_component(file ${msg} NAME ABSOLUTE)
    get_filename_component(dir1 ${msg} DIRECTORY ABSOLUTE)
    get_filename_component(name ${msg} NAME_WE)
    cmake_path(GET dir1 PARENT_PATH dir2)

    # snake case name
    string(REGEX REPLACE "([A-Z][a-z]+)" "_\\1" snake_case_name "${name}")
    string(TOLOWER "${snake_case_name}" snake_case_name)
    string(REGEX REPLACE "_(.*)" "\\1" snake_case_name "${snake_case_name}")

    message(STATUS "snake_case_name: ${snake_case_name}")
    message(STATUS "package: ${package}")
    message(STATUS "name: ${name}")
    message(STATUS "file: ${file}")

    set(output ${CMAKE_CURRENT_BINARY_DIR}/msg/detail/${snake_case_name}__functions.c)
    message(STATUS "output: ${output}")

    list(APPEND ${sources} ${output})
    add_custom_command(
      OUTPUT ${output}
      COMMAND rosidl generate -o ${CMAKE_CURRENT_BINARY_DIR} -t c ${package} ${msg}
      WORKING_DIRECTORY ${ZEPHYR_CURRENT_MODULE_DIR}
    )
  endforeach()
  message(STATUS "sources internal: ${sources}")
  set(${sources} ${${sources}} PARENT_SCOPE)
endfunction()

set(sources)
add_package("unique_identifier_msgs" sources)

message(STATUS "sources: ${sources}")

zephyr_library_sources(
  ${sources}
)

include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}
)

zephyr_library_include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}/..
)

endif()
