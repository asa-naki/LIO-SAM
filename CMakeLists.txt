cmake_minimum_required(VERSION 3.5)
project(lio_sam)

#Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()


#Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release)
endif()

set(BOOSTDIRS "/usr/lib/x86_64-linux-gnu/cmake/Boost-1.71.0/BoostConfig.cmake")

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(Boost REQUIRED iostreams)
find_package(rclpy REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(tf2_sensor_msgs REQUIRED)
find_package(tf2_eigen REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(OpenCV REQUIRED)
find_package(GTSAM REQUIRED)
find_package(Eigen3 REQUIRED)

find_package(PCL REQUIRED)



include_directories(
  include
)

rosidl_generate_interfaces(${PROJECT_NAME} "msg/CloudInfo.msg" DEPENDENCIES std_msgs sensor_msgs)

add_executable(${PROJECT_NAME}_featureExtraction src/featureExtraction.cpp)
ament_target_dependencies(${PROJECT_NAME}_featureExtraction rclcpp rclpy std_msgs sensor_msgs geometry_msgs nav_msgs tf2 tf2_ros tf2_eigen tf2_sensor_msgs tf2_geometry_msgs OpenCV PCL)
rosidl_target_interfaces(${PROJECT_NAME}_featureExtraction ${PROJECT_NAME} "rosidl_typesupport_cpp")
target_link_libraries(${PROJECT_NAME}_featureExtraction gtsam ${PCL_LIBRARIES})

add_executable(${PROJECT_NAME}_imageProjection src/imageProjection.cpp)
ament_target_dependencies(${PROJECT_NAME}_imageProjection rclcpp rclpy std_msgs sensor_msgs geometry_msgs nav_msgs tf2 tf2_ros tf2_eigen tf2_sensor_msgs tf2_geometry_msgs OpenCV PCL)
target_link_libraries(${PROJECT_NAME}_imageProjection gtsam ${PCL_LIBRARIES})
rosidl_target_interfaces(${PROJECT_NAME}_imageProjection ${PROJECT_NAME} "rosidl_typesupport_cpp")


add_executable(${PROJECT_NAME}_imuPreintegration src/imuPreintegration.cpp)
ament_target_dependencies(${PROJECT_NAME}_imuPreintegration rclcpp rclpy std_msgs sensor_msgs geometry_msgs nav_msgs tf2 tf2_ros tf2_eigen tf2_sensor_msgs tf2_geometry_msgs OpenCV PCL GTSAM Eigen)
target_link_libraries(${PROJECT_NAME}_imuPreintegration gtsam ${PCL_LIBRARIES})
rosidl_target_interfaces(${PROJECT_NAME}_imuPreintegration ${PROJECT_NAME} "rosidl_typesupport_cpp")

add_executable(${PROJECT_NAME}_mapOptimization src/mapOptmization.cpp)
ament_target_dependencies(${PROJECT_NAME}_mapOptimization rclcpp rclpy std_msgs sensor_msgs geometry_msgs nav_msgs visualization_msgs tf2 tf2_ros tf2_eigen tf2_sensor_msgs tf2_geometry_msgs OpenCV PCL GTSAM)
target_link_libraries(${PROJECT_NAME}_mapOptimization gtsam ${PCL_LIBRARIES})
rosidl_target_interfaces(${PROJECT_NAME}_mapOptimization ${PROJECT_NAME} "rosidl_typesupport_cpp")

install(
  DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}/
)

install(
  DIRECTORY config
  DESTINATION share/${PROJECT_NAME}/
)

install(
  TARGETS ${PROJECT_NAME}_imageProjection
  DESTINATION lib/${PROJECT_NAME}
)

install(
  TARGETS ${PROJECT_NAME}_imuPreintegration
  DESTINATION lib/${PROJECT_NAME}
)

install(
  TARGETS ${PROJECT_NAME}_featureExtraction
  DESTINATION lib/${PROJECT_NAME}
)

install(
  TARGETS ${PROJECT_NAME}_mapOptimization
  DESTINATION lib/${PROJECT_NAME}
)

install(
  DIRECTORY "include/"
  DESTINATION include
)

ament_export_include_directories(include)
ament_export_dependencies(Boost)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # uncomment the line when a copyright and license is not present in all source files
  #set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # uncomment the line when this package is not in a git repo
  #set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()

