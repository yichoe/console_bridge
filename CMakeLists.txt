cmake_minimum_required(VERSION 2.8)
project(console_bridge)

include(GNUInstallDirs)

if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# If compiler support symbol visibility, enable it.
include(CheckCCompilerFlag)
check_c_compiler_flag(-fvisibility=hidden HAS_VISIBILITY)
if (HAS_VISIBILITY)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
endif()

find_package(catkin REQUIRED)
find_package(Boost COMPONENTS system thread REQUIRED)

if(MSVC OR MSVC90 OR MSVC10)
  set(MSVC ON)
endif (MSVC OR MSVC90 OR MSVC10)

if(MSVC)
  add_definitions(-DBOOST_ALL_NO_LIB)
endif(MSVC)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  DEPENDS Boost
)

include_directories(include ${Boost_INCLUDE_DIRS})

if(NOT DEFINED BUILD_SHARED_LIBS)
  option(BUILD_SHARED_LIBS "Build dynamically-linked binaries" ON)
endif()

add_library(${PROJECT_NAME} src/console.cpp)
target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES})
set_target_properties(${PROJECT_NAME} PROPERTIES SOVERSION 0.2)

if (NOT MSVC)
  set(PKG_DESC "Console Bridge")
  set(PKG_CB_LIBS "-l${PROJECT_NAME}")
  set(pkg_conf_file "${CMAKE_CURRENT_SOURCE_DIR}/console_bridge.pc")
  configure_file("${pkg_conf_file}.in" "${CMAKE_BINARY_DIR}/${pkg_conf_file}" @ONLY)
  install(FILES ${CMAKE_BINARY_DIR}/${pkg_conf_file}
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig/ COMPONENT pkgconfig)
endif()

install(TARGETS ${PROJECT_NAME}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h")
