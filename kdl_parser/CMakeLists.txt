cmake_minimum_required(VERSION 3.0)

project(kdl_parser VERSION 1.13.0)

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

include_directories(include ${orocos_kdl_INCLUDE_DIRS} ${TinyXML_INCLUDE_DIRS} ${TinyXML2_INCLUDE_DIRS})

add_compile_options(-std=c++11)

add_library(${PROJECT_NAME} src/kdl_parser.cpp)
target_link_libraries(${PROJECT_NAME}
  ${TinyXML_LIBRARIES} ${TinyXML2_LIBRARIES} ${orocos_kdl_LIBRARIES}
)

if(urdf_FOUND)
  target_link_libraries(${PROJECT_NAME} ${urdf_LIBRARIES})
else()
  target_link_libraries(${PROJECT_NAME} ${urdfdom_LIBRARIES})
endif()

if(WIN32)
  target_compile_definitions(${PROJECT_NAME} PRIVATE "KDL_PARSER_BUILDING_DLL")
endif()

add_executable(check_kdl_parser src/check_kdl_parser.cpp )
target_link_libraries(check_kdl_parser ${PROJECT_NAME})

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
