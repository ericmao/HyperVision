cmake_minimum_required(VERSION 3.16 FATAL_ERROR)
project(HyperVision)
set(CMAKE_CXX_STANDARD 17)

# Platform-specific settings
if(APPLE)
    # macOS specific settings for ARM64 compatibility
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-strict-aliasing -fno-omit-frame-pointer")
    # Disable some optimizations that might cause issues on ARM64
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2")
    # Add memory alignment flags
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -falign-functions=16 -falign-loops=16")
endif()

# Find required packages
find_package(PkgConfig REQUIRED)

# Find gflags
find_library(GFLAGS_LIBRARIES NAMES gflags PATHS /opt/homebrew/lib)
find_path(GFLAGS_INCLUDE_DIRS NAMES gflags/gflags.h PATHS /opt/homebrew/include)
if(NOT GFLAGS_LIBRARIES OR NOT GFLAGS_INCLUDE_DIRS)
    message(FATAL_ERROR "gflags not found. Please install it via: brew install gflags")
endif()

# Try pkg-config for libpcap first, fall back to find_library on macOS
pkg_check_modules(PCAP libpcap)
if(NOT PCAP_FOUND)
    if(APPLE)
        # On macOS, try to find libpcap via find_library
        find_library(PCAP_LIBRARIES NAMES pcap)
        find_path(PCAP_INCLUDE_DIRS NAMES pcap/pcap.h)
        if(PCAP_LIBRARIES AND PCAP_INCLUDE_DIRS)
            set(PCAP_FOUND TRUE)
        else()
            message(FATAL_ERROR "libpcap not found. Please install it via: brew install libpcap")
        endif()
    else()
        message(FATAL_ERROR "libpcap not found")
    endif()
endif()

# Find PcapPlusPlus
find_package(PkgConfig)
pkg_check_modules(PCPP pcapplusplus)

# Find Boost
find_package(Boost REQUIRED)

# Set include directories
include_directories(${GFLAGS_INCLUDE_DIRS})
include_directories(${PCAP_INCLUDE_DIRS})
include_directories(${Boost_INCLUDE_DIRS})
if(PCPP_FOUND)
    include_directories(${PCPP_INCLUDE_DIRS})
    # Also add the parent directory for PcapPlusPlus
    include_directories(/opt/homebrew/Cellar/pcapplusplus/25.05/include)
endif()

# Add library search paths
link_directories(/opt/homebrew/lib)

add_subdirectory(flow_construct)
add_subdirectory(dataset_construct)
add_subdirectory(packet_parse)
add_subdirectory(graph_analyze)

aux_source_directory(. DIR_SRCS)
add_executable(${PROJECT_NAME} "${DIR_SRCS}")

target_link_libraries(${PROJECT_NAME} ${GFLAGS_LIBRARIES})
target_link_libraries(${PROJECT_NAME} ${PCAP_LIBRARIES})
target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES})
if(PCPP_FOUND)
    target_link_libraries(${PROJECT_NAME} ${PCPP_LIBRARIES})
endif()
target_link_libraries(${PROJECT_NAME} graph_analysis)
target_link_libraries(${PROJECT_NAME} flow_construct)
target_link_libraries(${PROJECT_NAME} dataset_construct)
target_link_libraries(${PROJECT_NAME} packet_parse)
