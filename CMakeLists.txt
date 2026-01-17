cmake_minimum_required(VERSION 3.30)

project(pcb2laser
    VERSION 0.0.1
    LANGUAGES C CXX
)

option(ENABLE_DEBUG "Enable debug mode" ON)

if(NOT ENABLE_DEBUG)
    add_compile_definitions(NDEBUG)
endif()

# ----------------------------------
# C++ standard
# ----------------------------------
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(Boost_DEBUG ON)
# ----------------------------------
# Compiler flags
# ----------------------------------
include(CheckCXXCompilerFlag)

check_cxx_compiler_flag(-fext-numeric-literals HAS_EXT_NUMERIC_LITERALS)
if(HAS_EXT_NUMERIC_LITERALS)
    add_compile_options(-fext-numeric-literals)
endif()

add_compile_options(-lboost_program_options)

# Warnings
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_compile_options(-Wall -Wextra)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "IBM")
    add_compile_options(-qfullpath)
endif()

# ----------------------------------
# config.h
# ----------------------------------
configure_file(
    ${CMAKE_SOURCE_DIR}/config.h.in
    ${CMAKE_BINARY_DIR}/config.h
)

# ----------------------------------
# Boost
# ----------------------------------
find_package(Boost REQUIRED
    COMPONENTS
        program_options
        filesystem
)

# ----------------------------------
# pkg-config
# ----------------------------------
find_package(PkgConfig REQUIRED)

pkg_check_modules(GLIBMM REQUIRED glibmm-2.4>=2.8)
pkg_check_modules(GDKMM REQUIRED gdkmm-2.4>=2.8)
pkg_check_modules(GERBV REQUIRED libgerbv>=2.1.0)
pkg_check_modules(RSVG REQUIRED librsvg-2.0>=2.0)

# ----------------------------------
# Optional librsvg features
# ----------------------------------
include(CheckSymbolExists)

set(CMAKE_REQUIRED_INCLUDES ${RSVG_INCLUDE_DIRS})
set(CMAKE_REQUIRED_LIBRARIES ${RSVG_LIBRARIES})

check_symbol_exists(
    rsvg_handle_render_document
    "librsvg/rsvg.h"
    HAVE_RSVG_HANDLE_RENDER_DOCUMENT
)

check_symbol_exists(
    rsvg_handle_get_intrinsic_size_in_pixels
    "librsvg/rsvg.h"
    HAVE_RSVG_HANDLE_GET_INTRINSIC_SIZE_IN_PIXELS
)

# ----------------------------------
# GEOS (optional)
# ----------------------------------
find_package(GEOS 3.8.1 QUIET)

if(GEOS_FOUND)
    message(STATUS "Found GEOS++")
    add_compile_definitions(
        USE_UNSTABLE_GEOS_CPP_API
        GEOS_VERSION="${GEOS_VERSION}"
    )
else()
    message(STATUS "GEOS not found, using Boost geometry only")
endif()

# ----------------------------------
# Executable
# ----------------------------------
add_executable(pcb2laser
    src/main.cpp
    src/autoleveller.cpp
    src/backtrack.cpp
    src/bg_helpers.cpp
    src/bg_operators.cpp
    src/board.cpp
    src/common.cpp
    src/drill.cpp
    src/eulerian_paths.cpp
    src/geos_helpers.cpp
    src/gerberimporter.cpp
    src/layer.cpp
    src/merge_near_points.cpp
    src/ngc_exporter.cpp
    src/options.cpp
    src/outline_bridges.cpp
    src/path_finding.cpp
    src/segmentize.cpp
    src/surface_vectorial.cpp
    src/svg_writer.cpp
    src/segment_tree.cpp
    src/tile.cpp
    src/trim_paths.cpp
    src/voronoi.cpp
)

target_include_directories(pcb2laser PRIVATE
    ${CMAKE_BINARY_DIR}
    ${CMAKE_SOURCE_DIR}/src/include
    ${Boost_INCLUDE_DIRS}
    ${GLIBMM_INCLUDE_DIRS}
    ${GDKMM_INCLUDE_DIRS}
    ${GERBV_INCLUDE_DIRS}
    ${RSVG_INCLUDE_DIRS}
)

target_link_libraries(pcb2laser LINK_PUBLIC
    ${Boost_LIBRARIES}
    ${GLIBMM_LIBRARIES}
    ${GDKMM_LIBRARIES}
    ${GERBV_LIBRARIES}
    ${RSVG_LIBRARIES}
)

if(GEOS_FOUND)
    target_include_directories(pcb2laser PRIVATE ${GEOS_INCLUDE_DIRS})
    target_link_libraries(pcb2laser PRIVATE ${GEOS_LIBRARIES})
endif()

