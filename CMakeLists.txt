cmake_minimum_required(VERSION 3.0.2)
project(ed_tutorials)

find_package(catkin REQUIRED COMPONENTS
  ed
  ed_gui_server
  ed_localization
  ed_navigation
  ed_object_models
  ed_perception
  ed_perception_models
  ed_rviz_plugins
  ed_sensor_integration
)

catkin_package()

include_directories(
  ${catkin_INCLUDE_DIRS}
)
