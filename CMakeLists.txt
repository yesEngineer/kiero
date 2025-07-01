cmake_minimum_required(VERSION 3.14)
project(kiero VERSION 1.2.12 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Include FetchContent module
include(FetchContent)

# Fetch MinHook
FetchContent_Declare(
    minhook
    GIT_REPOSITORY https://github.com/TsudaKageyu/minhook.git
    GIT_TAG master
    GIT_SHALLOW TRUE
)

# Make MinHook available
FetchContent_MakeAvailable(minhook)

# Create the kiero static library
add_library(kiero STATIC
    kiero.cpp
    kiero.h
)

# Set target properties
set_target_properties(kiero PROPERTIES
    OUTPUT_NAME "kiero"
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
)

# Include directories
target_include_directories(kiero PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
)

# Link libraries
target_link_libraries(kiero PUBLIC minhook)

# Windows-specific libraries
if(WIN32)
    target_link_libraries(kiero PUBLIC
        d3d9
        d3d10
        d3d11
        d3d12
        dxgi
        opengl32
    )
endif()

# Compiler-specific options
if(MSVC)
    target_compile_options(kiero PRIVATE /W4)
else()
    target_compile_options(kiero PRIVATE -Wall -Wextra -Wpedantic)
endif()

# Installation
install(TARGETS kiero
    EXPORT kieroTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)

install(FILES kiero.h
    DESTINATION include
)

# Export targets for other projects to use
install(EXPORT kieroTargets
    FILE kieroTargets.cmake
    NAMESPACE kiero::
    DESTINATION lib/cmake/kiero
)

# Create config file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    kieroConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/kieroConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/kieroConfig.cmake"
    INSTALL_DESTINATION lib/cmake/kiero
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/kieroConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/kieroConfigVersion.cmake"
    DESTINATION lib/cmake/kiero
)