# Copyright 2011, 2020, Florent Lamiraux, Guilhem Saurel, CNRS
#
# This file is part of hpp. This software is free software: you can redistribute
# it and/or modify it under the terms of the GNU Lesser General Public License
# as published by the Free Software Foundation, either version 3 of the License,
# or (at your option) any later version.
#
# This software is distributed in the hope that it will be useful, but WITHOUT
# ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
# FOR A PARTICULAR PURPOSE.  See the GNU General Lesser Public License for more
# details.  You should have received a copy of the GNU Lesser General Public
# License along with dynamic-graph. If not, see <http://www.gnu.org/licenses/>.

cmake_minimum_required(VERSION 3.10)

set(PROJECT_NAME hpp-template-corba)
set(PROJECT_DESCRIPTION "Template corba server")

set(PROJECT_USE_CMAKE_EXPORT TRUE)
set(CUSTOM_HEADER_DIR "hpp/corba/template")

# Check if the submodule cmake have been initialized
set(JRL_CMAKE_MODULES "${CMAKE_CURRENT_LIST_DIR}/cmake")
if(EXISTS "${JRL_CMAKE_MODULES}/base.cmake")
  message(STATUS "JRL cmakemodules found in 'cmake/' git submodule")
else()
  find_package(jrl-cmakemodules QUIET CONFIG)
  if(jrl-cmakemodules_FOUND)
    get_property(
      JRL_CMAKE_MODULES
      TARGET jrl-cmakemodules::jrl-cmakemodules
      PROPERTY INTERFACE_INCLUDE_DIRECTORIES)
    message(STATUS "JRL cmakemodules found on system at ${JRL_CMAKE_MODULES}")
  elseif(${CMAKE_VERSION} VERSION_LESS "3.14.0")
    message(
      FATAL_ERROR
        "\nCan't find jrl-cmakemodules. Please either:\n"
        "  - use git submodule: 'git submodule update --init'\n"
        "  - or install https://github.com/jrl-umi3218/jrl-cmakemodules\n"
        "  - or upgrade your CMake version to >= 3.14 to allow automatic fetching\n"
    )
  else()
    message(STATUS "JRL cmakemodules not found. Let's fetch it.")
    include(FetchContent)
    FetchContent_Declare(
      "jrl-cmakemodules"
      GIT_REPOSITORY "https://github.com/jrl-umi3218/jrl-cmakemodules.git")
    FetchContent_MakeAvailable("jrl-cmakemodules")
    FetchContent_GetProperties("jrl-cmakemodules" SOURCE_DIR JRL_CMAKE_MODULES)
  endif()
endif()

include("${JRL_CMAKE_MODULES}/hpp.cmake")
include("${JRL_CMAKE_MODULES}/idl.cmake")

compute_project_args(PROJECT_ARGS LANGUAGES CXX)
project(${PROJECT_NAME} ${PROJECT_ARGS})

# Defines paths.
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

# Trigger dependency to jrl-mal
add_required_dependency("omniORB4")
option(HPP_CORBA_ENABLE_HPP_UTIL OFF "Use hpp-util to log messages.")
if(HPP_CORBA_ENABLE_HPP_UTIL)
  add_project_dependency(hpp-util REQUIRED)
endif(HPP_CORBA_ENABLE_HPP_UTIL)

set(${PROJECT_NAME}_HEADERS
    include/${CUSTOM_HEADER_DIR}/doc.hh include/${CUSTOM_HEADER_DIR}/server.hh
    include/${CUSTOM_HEADER_DIR}/server.hxx)

set(${PROJECT_NAME}_SOURCES src/server.cc)

add_library(${PROJECT_NAME} SHARED ${${PROJECT_NAME}_SOURCES}
                                   ${${PROJECT_NAME}_HEADERS})
target_include_directories(
  ${PROJECT_NAME} PUBLIC $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
pkg_config_use_dependency(${PROJECT_NAME} omniORB4)

if(HPP_CORBA_ENABLE_HPP_UTIL)
  target_link_libraries(${PROJECT_NAME} hpp-util::hpp-util)
endif(HPP_CORBA_ENABLE_HPP_UTIL)

install(
  TARGETS ${PROJECT_NAME}
  EXPORT ${TARGETS_EXPORT_NAME}
  DESTINATION lib)

pkg_config_append_libs(${PROJECT_NAME})

add_subdirectory(doc)
add_subdirectory(include)
add_subdirectory(tests)

install(FILES package.xml DESTINATION share/${PROJECT_NAME})
