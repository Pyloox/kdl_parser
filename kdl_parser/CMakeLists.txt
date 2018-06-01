cmake_minimum_required(VERSION 3.0)

project(kdl_parser VERSION 1.13.0)

find_package(catkin QUIET
  COMPONENTS rosconsole cmake_modules
)

if(NOT catkin_FOUND)
  set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
endif()

find_package(urdf QUIET)
if(urdf_FOUND)
  add_definitions(-DHAS_URDF)
  include_directories(${urdf_INCLUDE_DIRS})
else()
  find_package(urdfdom REQUIRED)
  include_directories(${urdfdom_INCLUDE_DIRS})
endif()

find_package(orocos_kdl REQUIRED)
find_package(TinyXML REQUIRED)
find_package(TinyXML2 REQUIRED)

find_package(rosconsole QUIET)
if(rosconsole_FOUND)
  add_definitions(-DHAS_ROS)
endif()

include_directories(include ${orocos_kdl_INCLUDE_DIRS} ${TinyXML_INCLUDE_DIRS} ${TinyXML2_INCLUDE_DIRS})

add_compile_options(-std=c++11)

if(catkin_FOUND)
  link_directories(${catkin_LIBRARY_DIRS})
  include_directories(${catkin_INCLUDE_DIRS})

  catkin_package(
    LIBRARIES ${PROJECT_NAME} ${orocos_kdl_LIBRARIES}
    INCLUDE_DIRS include
    CATKIN_DEPENDS rosconsole urdf
    DEPENDS orocos_kdl TinyXML TinyXML2
  )
endif()

add_library(${PROJECT_NAME} src/kdl_parser.cpp)
target_link_libraries(${PROJECT_NAME}
  ${TinyXML_LIBRARIES} ${TinyXML2_LIBRARIES} ${orocos_kdl_LIBRARIES}
)

if(urdf_FOUND)
  target_link_libraries(${PROJECT_NAME} ${urdf_LIBRARIES})
else()
  target_link_libraries(${PROJECT_NAME} ${urdfdom_LIBRARIES})
endif()

if(catkin_FOUND)
  target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES})
endif()

if(WIN32)
  target_compile_definitions(${PROJECT_NAME} PRIVATE "KDL_PARSER_BUILDING_DLL")
endif()

add_executable(check_kdl_parser src/check_kdl_parser.cpp )
target_link_libraries(check_kdl_parser ${PROJECT_NAME})

if(catkin_FOUND AND CATKIN_ENABLE_TESTING)
  find_package(catkin REQUIRED COMPONENTS roscpp rostest)
  add_rostest_gtest(test_kdl_parser test/test_kdl_parser.launch test/test_kdl_parser.cpp )
  target_link_libraries(test_kdl_parser ${PROJECT_NAME})

  add_rostest_gtest(test_inertia_rpy test/test_inertia_rpy.launch test/test_inertia_rpy.cpp )
  target_link_libraries(test_inertia_rpy ${PROJECT_NAME})
endif()

if(catkin_FOUND)
  # How does CATKIN do this?
  #rosbuild_add_rostest(${PROJECT_SOURCE_DIR}/test/test_kdl_parser.launch)
  install(TARGETS ${PROJECT_NAME}
    DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})

  install(DIRECTORY include/${PROJECT_NAME}/
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})
else() 
  install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}Targets
          LIBRARY DESTINATION lib
          ARCHIVE DESTINATION lib
          INCLUDES DESTINATION include/${PROJECT_NAME}/)

  install(DIRECTORY include/${PROJECT_NAME}/
          DESTINATION include/${PROJECT_NAME})

  install(EXPORT ${PROJECT_NAME}Targets
          FILE ${PROJECT_NAME}Targets.cmake
          NAMESPACE ${PROJECT_NAME}::
          DESTINATION lib/cmake/${PROJECT_NAME})

  include(CMakePackageConfigHelpers)
  write_basic_package_version_file("${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${${PROJECT_NAME}_VERSION}
    COMPATIBILITY SameMajorVersion)
  install(FILES "cmake/${PROJECT_NAME}Config.cmake" "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION lib/cmake/${PROJECT_NAME})

endif()
