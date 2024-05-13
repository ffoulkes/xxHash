# Copyright 2024 Intel Corporation
# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.15)

project(xxhash VERSION 0.8.3 LANGUAGES C)

#-----------------------------------------------------------------------
# Modules
#-----------------------------------------------------------------------
include(CMakePackageConfigHelpers)
include(CMakePrintHelpers)
include(GNUInstallDirs)

#-----------------------------------------------------------------------
# Settings
#-----------------------------------------------------------------------
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

#-----------------------------------------------------------------------
# Build
#-----------------------------------------------------------------------
# object library
add_library(xxhash_o OBJECT
  xxh_x86dispatch.c
  xxh_x86dispatch.h
  xxhash.c
  xxhash.h
)

# shared library
add_library(xxhash SHARED
  $<TARGET_OBJECTS:xxhash_o>
)

target_include_directories(xxhash PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}/xxhash>
)

set_target_properties(xxhash PROPERTIES
  VERSION ${PROJECT_VERSION}
  SOVERSION ${PROJECT_VERSION_MAJOR}
)

# static library
add_library(xxhash_static STATIC
  $<TARGET_OBJECTS:xxhash_o>
)

set_target_properties(xxhash_static PROPERTIES OUTPUT_NAME xxhash)

#-----------------------------------------------------------------------
# Install
#-----------------------------------------------------------------------
set(EXPORTNAME xxHash)
set(EXPORTSTEM xxhash)

install(
  TARGETS xxhash
  EXPORT ${EXPORTNAME}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(
    FILES xxh3.h xxhash.h xxh_x86dispatch.h
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/xxhash
)

#-----------------------------------------------------------------------
# pkg-config file
#-----------------------------------------------------------------------
set(_pkgconfig_file ${CMAKE_CURRENT_BINARY_DIR}/pkgconfig/libxxhash.pc)

set(PREFIX ${CMAKE_INSTALL_PREFIX})
set(EXECPREFIX "\${prefix}/bin")
set(INCLUDEDIR "\${prefix}/include")
set(LIBDIR "\${prefix}/lib")
set(VERSION ${PROJECT_VERSION})

configure_file(libxxhash.pc.in ${_pkgconfig_file} @ONLY)

install(
  FILES ${_pkgconfig_file}
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)

#-----------------------------------------------------------------------
# CMake configuration file
#-----------------------------------------------------------------------
install(EXPORT ${EXPORTNAME}
  FILE ${EXPORTSTEM}-targets.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${EXPORTSTEM}
)

configure_package_config_file(
  ${EXPORTSTEM}-config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${EXPORTSTEM}-config.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${EXPORTSTEM}
)

install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${EXPORTSTEM}-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${EXPORTSTEM}-config-version.cmake
  DESTINATION
    ${CMAKE_INSTALL_LIBDIR}/cmake/${EXPORTSTEM}
)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${EXPORTSTEM}-config-version.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)
