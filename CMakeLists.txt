cmake_minimum_required(VERSION 2.8.12)

cmake_policy(SET CMP0048 NEW)

project(thundersvm VERSION 0.1.0 LANGUAGES C CXX)
set(USE_CUDA ON CACHE BOOL "Compile with CUDA")
set(USE_EIGEN OFF CACHE BOOL "Compile with Eigen")

find_package(Threads REQUIRED)

#find_package(MKL REQUIRED)
#include_directories(${MKL_INCLUDE_DIRS})
#link_directories(${MKL_LIBRARIES})
#target_link_libraries(<module>
#mkl_intel_lp64
#mkl_sequential
#mkl_core
#)
if (USE_CUDA)
    message("Compile with CUDA")
    find_package(CUDA REQUIRED QUIET)
    set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} -std=c++11 -Wno-deprecated-gpu-targets)
    set(LINK_LIBRARY ${CUDA_cusparse_LIBRARY})
else ()
    message("Compile without CUDA")
    find_package(OpenMP REQUIRED)
endif ()

if (USE_EIGEN)
    message("using Eigen for acceleration")
    include_directories(${PROJECT_SOURCE_DIR}/eigen)
endif ()


if (CMAKE_VERSION VERSION_LESS "3.1")
    add_compile_options("-std=c++11")
else ()
    set(CMAKE_CXX_STANDARD 11)
endif ()

set(COMMON_INCLUDES ${PROJECT_SOURCE_DIR}/include ${CMAKE_CURRENT_BINARY_DIR})


set(DATASET_DIR ${PROJECT_SOURCE_DIR}/dataset/)
configure_file(include/thundersvm/config.h.in config.h)

set(PROJECT_LIB_NAME ${PROJECT_NAME}-lib)
include_directories(${COMMON_INCLUDES})
add_subdirectory(${PROJECT_SOURCE_DIR}/src/thundersvm)


set(PROJECT_TEST_NAME ${PROJECT_NAME}-test)
add_subdirectory(${PROJECT_SOURCE_DIR}/src/test)
add_custom_target(runtest
        COMMAND ${PROJECT_TEST_NAME})
