cmake_minimum_required(VERSION 3.12)

project(diffvg VERSION 0.0.1 DESCRIPTION "Differentiable Vector Graphics")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(WIN32)
    find_package(Python 3.6 COMPONENTS Development REQUIRED)
else()
    find_package(Python 3.7 COMPONENTS Development REQUIRED)
endif()

add_subdirectory(pybind11)

option(DIFFVG_CUDA "Build diffvg with GPU code path?" ON)

if(DIFFVG_CUDA)
    message(STATUS "Build with CUDA support")
    find_package(CUDA 10 REQUIRED)
    set(CMAKE_CUDA_STANDARD 11)
    if(NOT WIN32)
        set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -std=c++11")
    endif()
else()
    message(STATUS "Build without CUDA support")
    find_package(Thrust REQUIRED)
endif()

include_directories(${PYTHON_INCLUDE_PATH})
find_package(PythonLibs REQUIRED)
include_directories(${PYTHON_INCLUDE_PATH})
include_directories(${PYTHON_INCLUDE_DIRS})
include_directories(pybind11/include)

if(DIFFVG_CUDA)
    link_directories(${CUDA_LIBRARIES})
else()
    include_directories(${THRUST_INCLUDE_DIR})
endif()

if(NOT MSVC)
  add_compile_options(-Wall -g -O3 -fvisibility=hidden -Wno-unknown-pragmas)
else()
  add_compile_options(/Wall /Zi)
  add_link_options(/DEBUG)
endif()

if(NOT DIFFVG_CUDA)
    add_compile_options("-DTHRUST_DEVICE_SYSTEM=THRUST_DEVICE_SYSTEM_CPP")
endif()

set(SRCS atomic.h
         color.h
         cdf.h
         cuda_utils.h
         diffvg.h
         edge_query.h
         filter.h
         matrix.h
         parallel.h
         pcg.h
         ptr.h
         sample_boundary.h
         scene.h
         shape.h
         solve.h
         vector.h
         within_distance.h
         winding_number.h
         atomic.cpp
         color.cpp
         diffvg.cpp
         parallel.cpp
         scene.cpp
         shape.cpp)

if(DIFFVG_CUDA)
    add_compile_definitions(COMPILE_WITH_CUDA)
    set_source_files_properties(
        diffvg.cpp
        scene.cpp
        PROPERTIES CUDA_SOURCE_PROPERTY_FORMAT OBJ)

    cuda_add_library(diffvg MODULE ${SRCS})
else()
    add_library(diffvg MODULE ${SRCS})
endif()

if(WIN32)
    # Windows: ensure proper module extension is `.pyd`
    target_link_libraries(diffvg pybind11::module)
    set_target_properties(diffvg PROPERTIES
                          PREFIX ""
                          SUFFIX ".pyd")
else()
    # On UNIX or macOS, we use standard suffixes
    if(APPLE)
        set(DYNAMIC_LOOKUP "-undefined dynamic_lookup")
    endif()
    target_link_libraries(diffvg ${DYNAMIC_LOOKUP})
endif()

# Ensure output directory is correct for Windows
if(WIN32)
    set_target_properties(diffvg PROPERTIES
        LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/diffvg")
endif()

# Set RPATH for shared libraries
set_target_properties(diffvg PROPERTIES SKIP_BUILD_RPATH FALSE)
set_target_properties(diffvg PROPERTIES BUILD_WITH_INSTALL_RPATH TRUE)
if(UNIX AND NOT APPLE)
    set_target_properties(diffvg PROPERTIES INSTALL_RPATH "$ORIGIN")
elseif(APPLE)
    set_target_properties(diffvg PROPERTIES INSTALL_RPATH "@loader_path")
endif()

set_property(TARGET diffvg PROPERTY CXX_STANDARD 11)
set_target_properties(diffvg PROPERTIES PREFIX "")

# Still enable assertion in release mode
string(REPLACE "/DNDEBUG" "" CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")
string(REPLACE "-DNDEBUG" "" CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")

# Optional TensorFlow support
if(NOT WIN32)
    find_package(TensorFlow)
    if(TensorFlow_FOUND)
        add_subdirectory(pydiffvg_tensorflow/custom_ops)
    else()
        message(INFO " Building without TensorFlow support (not found)")
    endif()
endif()