set(execName JetsonZedTemplate)
cmake_minimum_required(VERSION 3.10)
project(${execName})

set(ZED_DIR "/usr/local/zed/")
list(APPEND CMAKE_PREFIX_PATH ${ZED_DIR})

option(LINK_SHARED_ZED "Link with the ZED SDK shared executable" ON)

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0")
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -O0")

if (NOT LINK_SHARED_ZED AND MSVC)
    message(FATAL_ERROR "LINK_SHARED_ZED OFF : ZED SDK static libraries not available on Windows")
endif()

if(COMMAND cmake_policy)
    cmake_policy(SET CMP0003 OLD)
    cmake_policy(SET CMP0015 OLD)
endif(COMMAND cmake_policy)

set(EXECUTABLE_OUTPUT_PATH ".")

find_package(zed REQUIRED)
find_package(OpenCV REQUIRED)
find_package(CUDA ${ZED_CUDA_VERSION} EXACT REQUIRED)

include_directories("/usr/include/c++/7")
include_directories(${CUDA_INCLUDE_DIRS})
include_directories(${ZED_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

link_directories(${ZED_LIBRARY_DIR})
link_directories(${CUDA_LIBRARY_DIRS})

# Add files to executable here:
add_executable(${execName} main.cpp)

add_definitions(-std=c++11 -g -O3)

if (LINK_SHARED_ZED)
    set(ZED_LIBS ${ZED_LIBRARIES} ${CUDA_CUDA_LIBRARY} ${CUDA_CUDART_LIBRARY} ${CUDA_NPP_LIBRARIES_ZED})
else()
    set(ZED_LIBS ${ZED_STATIC_LIBRARIES} ${CUDA_CUDA_LIBRARY} ${CUDA_LIBRARY})
endif()

set(CUDA_LIBS ${CUDA_npp_LIBRARY})
target_link_libraries(${execName} ${ZED_LIBS} ${OpenCV_LIBS} ${CUDA_LIBS})