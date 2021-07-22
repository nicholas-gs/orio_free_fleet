cmake_minimum_required(VERSION 3.5)
project(orio_magni_ff_server)

find_package(ament_cmake QUIET)

if (ament_cmake_FOUND)

  install(
    DIRECTORY launch
    DESTINATION share/${PROJECT_NAME}
  )

else()
  message("ament_cmake not found so skipping this ROS2 package")
endif()

ament_package()
