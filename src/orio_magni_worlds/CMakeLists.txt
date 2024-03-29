cmake_minimum_required(VERSION 3.5)
project(orio_magni_worlds)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake QUIET)

if (ament_cmake_FOUND)

  install(
    DIRECTORY worlds/
    DESTINATION share/${PROJECT_NAME}/worlds
  )

  install(
    DIRECTORY adapters/
    DESTINATION share/${PROJECT_NAME}/adapters
  )

  install(
    DIRECTORY rviz/
    DESTINATION share/${PROJECT_NAME}/rviz
  )

  file(GLOB_RECURSE traffic_editor_paths "worlds/*.building.yaml")

  foreach(path ${traffic_editor_paths})
    # Get the output world name
    string(REGEX REPLACE "\\.[^.]*\.[^.]*$" "" no_extension_path ${path})
    string(REGEX MATCH "[^\/]+$" world_name  ${no_extension_path})

    set(map_path ${path})
    set(output_world_name ${world_name})
    set(output_dir ${CMAKE_CURRENT_BINARY_DIR}/maps/${output_world_name})
    set(output_world_path ${output_dir}/${output_world_name}.world)
    set(output_model_dir ${output_dir}/models)

    ##############################################################################
    # first, generate the world
    message("BUILDING WORLDFILE WITH COMMAND: ros2 run rmf_building_map_tools building_map_generator gazebo ${map_path} ${output_world_path} ${output_model_dir}")
    if (NO_DOWNLOAD_MODELS)
      add_custom_command(
        DEPENDS ${map_path}
        COMMAND ros2 run rmf_building_map_tools building_map_generator gazebo ${map_path} ${output_world_path} ${output_model_dir}
        OUTPUT ${output_world_path}
      )
    else()
      message("DOWNLOADING MODELS WITH COMMAND: ros2 run rmf_building_map_tools building_map_model_downloader ${map_path}")
      add_custom_command(
        DEPENDS ${map_path}
        COMMAND ros2 run rmf_building_map_tools building_map_generator gazebo ${map_path} ${output_world_path} ${output_model_dir}
        COMMAND ros2 run rmf_building_map_tools building_map_model_downloader ${map_path}
        OUTPUT ${output_world_path}
      )
    endif()

    add_custom_target(generate_${output_world_name} ALL
      DEPENDS ${output_world_path}
    )

    ##############################################################################
    # generate the navmesh and required files for crowd simulation for gz
    set(crowd_sim_config_resource ${output_dir}/config_resource/)

    add_custom_command(
      OUTPUT ${world_name}_crowdsim
      COMMAND ros2 run rmf_building_map_tools building_crowdsim ${map_path} ${crowd_sim_config_resource} ${output_world_path}
      DEPENDS ${output_world_path}
    )
    add_custom_target(generate_${world_name}_crowdsim ALL
      DEPENDS ${world_name}_crowdsim
    )  

    ##############################################################################
    ## Ign worlds

    # generate Ignition simulation world
    set(ign_output_dir ${CMAKE_CURRENT_BINARY_DIR}/maps/${output_world_name}_ign)
    set(ign_output_world_path ${ign_output_dir}/${output_world_name}.world)
    set(ign_output_model_dir ${ign_output_dir}/models)

    add_custom_command(
      OUTPUT ${ign_output_world_path}
      COMMAND ros2 run rmf_building_map_tools building_map_generator ignition ${map_path} ${ign_output_world_path} ${ign_output_model_dir}
      DEPENDS ${map_path}
    )

    add_custom_target(generate_${output_world_name}_ign ALL
      DEPENDS ${ign_output_world_path}
    )

    ##############################################################################
    # generate the navmesh and required files for crowd simulation
    add_custom_command(
      OUTPUT ${world_name}_crowdsim_ign
      COMMAND ros2 run rmf_building_map_tools building_crowdsim ${map_path} ${crowd_sim_config_resource} ${ign_output_world_path}
      DEPENDS ${ign_output_world_path}
    )
    add_custom_target(generate_${world_name}_crowdsim_ign ALL
      DEPENDS ${world_name}_crowdsim_ign
    )

    install(
      DIRECTORY ${ign_output_dir}
      DESTINATION share/${PROJECT_NAME}/maps
    )

    ##############################################################################
    # now, generate the nav graphs
    set(output_nav_graphs_dir ${output_dir}/nav_graphs/)
    set(output_nav_graphs_phony ${output_nav_graphs_dir}/phony)
    add_custom_command(
      OUTPUT ${output_nav_graphs_phony}
      COMMAND ros2 run rmf_building_map_tools building_map_generator nav ${map_path} ${output_nav_graphs_dir}
      DEPENDS ${map_path}
    )

    add_custom_target(generate_${output_world_name}_nav_graphs ALL
      DEPENDS ${output_nav_graphs_phony}
    )

    install(
      DIRECTORY ${output_dir}
      DESTINATION share/${PROJECT_NAME}/maps
    )
  endforeach()

  # Install launch files
  install(
    DIRECTORY ${output_dir}
    DESTINATION share/${PROJECT_NAME}/launch
  )

  ament_package()

else()
  message("ament_cmake not found so skipping this ROS2 package")
endif()
