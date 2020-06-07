cmake_minimum_required(VERSION 3.10)

project(voro++_2d VERSION 0.4.6 LANGUAGES CXX)
set(SOVERSION "0")

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CXX_FLAGS)
  #release comes with -O3 by default
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CXX_FLAGS)

########################################################################
# User input options                                                   #
########################################################################
option(BUILD_SHARED_LIBS "Build shared libs" ON)
include(GNUInstallDirs)

######################################
# Include the following subdirectory # 
######################################

file(GLOB VORO_2D_SOURCES src/*.cc)
file(GLOB NOT_VORO_2D_SOURCES src/v_base_wl_2d.cc src/voro++_2d.cc src/v_connect.cc src/*_test.cc src/*_old.cc)
list(REMOVE_ITEM VORO_2D_SOURCES ${NOT_VORO_2D_SOURCES})
add_library(voro++_2d ${VORO_2D_SOURCES})
set_target_properties(voro++_2d PROPERTIES 
  LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/src"
  SOVERSION ${SOVERSION})
install(TARGETS voro++_2d EXPORT VORO_2D_Targets LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
#for voro++_2d.hh. SYSTEM disables warnings
target_include_directories(voro++_2d SYSTEM PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src> $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

# Examples
file(GLOB EXAMPLE_SOURCES examples/*/*.cc) 
foreach(SOURCE ${EXAMPLE_SOURCES})
  string(REGEX REPLACE "^.*/([^/]*)\\.cc$" "\\1" PROGNAME "${SOURCE}")
  if (NOT PROGNAME STREQUAL ellipsoid) #ellipsoid is broken
    string(REGEX REPLACE "^.*/(examples/.*)/${PROGNAME}\\.cc$" "\\1" DIRNAME "${SOURCE}")
    add_executable(${PROGNAME} EXCLUDE_FROM_ALL ${SOURCE})
    target_link_libraries(${PROGNAME} PRIVATE voro++_2d)
    set_target_properties(${PROGNAME} PROPERTIES 
      RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${DIRNAME}" )
  endif()
endforeach(SOURCE)

# Installation
file(GLOB_RECURSE VORO_2D_HEADERS src/voro++_2d.hh)
install(FILES ${VORO_2D_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(FILES ${CMAKE_SOURCE_DIR}/man/voro++_2d.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)
# no external deps for we can use target file as config file
install(EXPORT VORO_2D_Targets FILE VORO2DConfig.cmake NAMESPACE VORO_2D:: DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/VORO_2D)
include(CMakePackageConfigHelpers)
write_basic_package_version_file("VORO2DConfigVersion.cmake" VERSION ${PROJECT_VERSION} COMPATIBILITY ExactVersion)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/VORO2DConfigVersion.cmake" DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/VORO_2D)

# Documentation
find_package(Doxygen)
if (DOXYGEN_FOUND)
  add_custom_target(doxygen COMMAND ${DOXYGEN_EXECUTABLE} src/Doxyfile 
    COMMENT "Build doxygen documentation")
endif (DOXYGEN_FOUND)
