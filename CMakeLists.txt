cmake_minimum_required(VERSION 3.5)

set(GKLIB_VERSION 1.0.0)

project(GKlib
  VERSION ${GKLIB_VERSION}
  LANGUAGES C)

include(GNUInstallDirs)

option(BUILD_SHARED_LIBS "Build shared libraries (.dll/.so) instead of static ones (.lib/.a)" OFF)

if(BUILD_SHARED_LIBS)
  set(LIBRARY_TYPE "SHARED")
else()
  set(LIBRARY_TYPE "STATIC")
endif()

get_filename_component(abs "${CMAKE_CURRENT_LIST_DIR}" ABSOLUTE)
set(GKLIB_PATH ${abs})
unset(abs)
include(GKlibSystem.cmake)

if(MSVC)
  file(GLOB win32_sources RELATIVE "win32" "*.c")
else(MSVC)
  set(win32_sources, "")
endif(MSVC)

add_library(GKlib ${LIBRARY_TYPE} ${GKlib_sources} ${win32_sources})

if(MSVC)
  target_include_directories(GKlib
    PRIVATE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/win32>
    INTERFACE
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  )
else()
  target_include_directories(GKlib
    PRIVATE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
    INTERFACE
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  )
endif()

if(UNIX)
  target_link_libraries(GKlib m)
endif(UNIX)

include(CMakePackageConfigHelpers)

configure_package_config_file("${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}Config.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
  INSTALL_DESTINATION "cmake/${PROJECT_NAME}"
)

write_basic_package_version_file("${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
  VERSION ${GKLIB_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(TARGETS GKlib
  EXPORT "${PROJECT_NAME}Targets"
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

install(FILES ${GKlib_includes}
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake" "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
  DESTINATION "cmake/"
)

install(EXPORT "${PROJECT_NAME}Targets"
  NAMESPACE "${PROJECT_NAME}::"
  DESTINATION "cmake/"
)

include_directories("test")
add_subdirectory("test")

