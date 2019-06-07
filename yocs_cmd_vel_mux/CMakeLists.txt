cmake_minimum_required(VERSION 2.8.3)
project(yocs_cmd_vel_mux)
find_package(catkin REQUIRED COMPONENTS roscpp pluginlib nodelet dynamic_reconfigure geometry_msgs)

# pkg-config support
find_package(PkgConfig)
if(MSVC)
        find_package(yaml-cpp REQUIRED)
else()
        pkg_search_module(yaml-cpp REQUIRED yaml-cpp)
        # todo: fall back to find_package if pkg_search_module does not find yaml-cpp
endif()

if(MSVC)
        if(NOT CMAKE_CXX_STANDARD)
                set(CMAKE_CXX_STANDARD 11)
        endif()
else()
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif()

if(NOT ${YAML_CPP_VERSION} VERSION_LESS "0.5")
add_definitions(-DHAVE_NEW_YAMLCPP)
endif()

# Dynamic reconfigure support
generate_dynamic_reconfigure_options(cfg/reload.cfg)

catkin_package(
   INCLUDE_DIRS include
   LIBRARIES ${PROJECT_NAME}_nodelet
   CATKIN_DEPENDS roscpp pluginlib nodelet dynamic_reconfigure geometry_msgs
   DEPENDS YAML_CPP
)
# todo: check if yaml-cpp would work, is it true YAML_CPP should be used instead

include_directories(include ${catkin_INCLUDE_DIRS} ${YAML_CPP_INCLUDE_DIRS})

# Nodelet library
add_library(${PROJECT_NAME}_nodelet src/cmd_vel_mux_nodelet.cpp src/cmd_vel_subscribers.cpp)
add_dependencies(${PROJECT_NAME}_nodelet geometry_msgs_gencpp)
add_dependencies(${PROJECT_NAME}_nodelet ${PROJECT_NAME}_gencfg)

# message(FATAL_ERROR "YAML-CPP libraries: ${YAML_CPP_INCLUDE_DIRS}, ${YAML_CPP_LIBRARIES}")
target_link_libraries(${PROJECT_NAME}_nodelet ${catkin_LIBRARIES} ${YAML_CPP_LIBRARIES})

install(TARGETS ${PROJECT_NAME}_nodelet
        DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)
install(DIRECTORY include/${PROJECT_NAME}/
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)
install(DIRECTORY plugins
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
install(DIRECTORY launch
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
install(DIRECTORY param
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
