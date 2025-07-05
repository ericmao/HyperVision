cmake_minimum_required(VERSION 3.10 FATAL_ERROR)
project(graph_analysis)
set(CMAKE_CXX_STANDARD 17)

# Handle OpenMP for different platforms
if(APPLE)
    # On macOS, try to find OpenMP via Homebrew
    find_package(OpenMP)
    if(OpenMP_CXX_FOUND)
        add_compile_options(${OpenMP_CXX_FLAGS})
    else()
        # If OpenMP is not found, we'll compile without it
        message(WARNING "OpenMP not found, compiling without OpenMP support")
    endif()
else()
    # On Linux, use the standard approach
    add_compile_options(-fopenmp)
    find_package(OpenMP)
    if(OpenMP_CXX_FOUND)
        target_link_libraries(${PROJECT_NAME} OpenMP::OpenMP_CXX)
    endif()
endif()

aux_source_directory(. DIR_LIB_SRCS)
add_library (${PROJECT_NAME} ${DIR_LIB_SRCS})

# Link OpenMP if found
if(OpenMP_CXX_FOUND)
    target_link_libraries(${PROJECT_NAME} OpenMP::OpenMP_CXX)
endif()

target_link_libraries(${PROJECT_NAME}
    armadillo
)

# Try to find z3, but don't fail if not found
find_package(PkgConfig)
pkg_check_modules(Z3 z3)
if(Z3_FOUND)
    target_link_libraries(${PROJECT_NAME} ${Z3_LIBRARIES})
    target_include_directories(${PROJECT_NAME} PRIVATE ${Z3_INCLUDE_DIRS})
else()
    message(WARNING "Z3 not found, some features may not work")
endif()
