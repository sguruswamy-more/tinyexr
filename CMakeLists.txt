cmake_minimum_required(VERSION 3.14)

project(tinyexr
  VERSION 1.0.0
  DESCRIPTION "Header-only C++ library to load and save OpenEXR (.exr) images"
  LANGUAGES CXX C
)

# ============================================================================
# Options
# ============================================================================
option(TINYEXR_BUILD_SAMPLE "Build sample/test executable" ON)
option(TINYEXR_USE_MINIZ "Use bundled miniz for compression" ON)

# ============================================================================
# Build miniz dependency (optional)
# ============================================================================
if(TINYEXR_USE_MINIZ)
  add_library(miniz STATIC deps/miniz/miniz.c deps/miniz/miniz.h)
  add_library(tinyexr::miniz ALIAS miniz)
  
  target_include_directories(miniz
    PUBLIC
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/deps/miniz>
      $<INSTALL_INTERFACE:include>
  )
  
  set_target_properties(miniz PROPERTIES
    POSITION_INDEPENDENT_CODE ON
    FOLDER "tinyexr/dependencies"
    EXPORT_NAME miniz
  )
endif()

# ============================================================================
# Main tinyexr library target
# ============================================================================
add_library(tinyexr tinyexr.cc tinyexr.h)
add_library(tinyexr::tinyexr ALIAS tinyexr)

target_include_directories(tinyexr
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
)

target_compile_features(tinyexr PUBLIC cxx_std_11)

# Link dependencies
if(TINYEXR_USE_MINIZ)
  target_link_libraries(tinyexr PUBLIC tinyexr::miniz)
  target_compile_definitions(tinyexr PUBLIC TINYEXR_USE_MINIZ=1)
endif()

if(CMAKE_DL_LIBS)
  target_link_libraries(tinyexr PRIVATE ${CMAKE_DL_LIBS})
endif()

set_target_properties(tinyexr PROPERTIES
  POSITION_INDEPENDENT_CODE ON
  CXX_STANDARD 11
  CXX_STANDARD_REQUIRED ON
  CXX_EXTENSIONS OFF
  VERSION ${PROJECT_VERSION}
  SOVERSION ${PROJECT_VERSION_MAJOR}
  EXPORT_NAME tinyexr
)

# Compiler warnings for development
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND NOT MSVC)
  target_compile_options(tinyexr PRIVATE
    -Weverything
    -Wno-padded
    -Wno-c++98-compat-pedantic
    -Wno-documentation
    -Wno-unused-member-function
  )
  if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 16)
    target_compile_options(tinyexr PRIVATE -Wno-unsafe-buffer-usage)
  endif()
elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
  target_compile_options(tinyexr PRIVATE -Wall -Wextra)
elseif(MSVC)
  target_compile_options(tinyexr PRIVATE /W4)
endif()

# ============================================================================
# Sample/Test executable
# ============================================================================
if(TINYEXR_BUILD_SAMPLE)
  add_executable(test_tinyexr test_tinyexr.cc)
  
  target_link_libraries(test_tinyexr PRIVATE tinyexr::tinyexr)
  
  if(WIN32)
    target_compile_definitions(test_tinyexr PRIVATE UNICODE _UNICODE)
    set_property(DIRECTORY PROPERTY VS_STARTUP_PROJECT test_tinyexr)
    
    if(CMAKE_VERSION VERSION_GREATER_EQUAL 3.8)
      set_target_properties(test_tinyexr PROPERTIES
        VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
      )
    endif()
  endif()
  
  set_target_properties(test_tinyexr PROPERTIES
    FOLDER "tinyexr"
  )
endif()

# ============================================================================
# Installation
# ============================================================================
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# Install library targets
install(TARGETS tinyexr
  EXPORT tinyexrTargets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Install miniz if used
if(TINYEXR_USE_MINIZ)
  install(TARGETS miniz
    EXPORT tinyexrTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  )
  install(FILES deps/miniz/miniz.h
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )
endif()

# Install headers
install(FILES tinyexr.h
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Export targets
install(EXPORT tinyexrTargets
  FILE tinyexrTargets.cmake
  NAMESPACE tinyexr::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tinyexr
)

# Create config-version file
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/tinyexrConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)

# Create config file
configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/tinyexrConfig.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/tinyexrConfig.cmake"
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tinyexr
)

# Install config files
install(FILES
  "${CMAKE_CURRENT_BINARY_DIR}/tinyexrConfig.cmake"
  "${CMAKE_CURRENT_BINARY_DIR}/tinyexrConfigVersion.cmake"
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tinyexr
)
