cmake_minimum_required(VERSION 3.15)

# CMP0091: MSVC runtime library flags are selected by CMAKE_MSVC_RUNTIME_LIBRARY
cmake_policy(SET CMP0091 NEW)

# Force vcpkg to use static libraries so only wxhelper.dll is produced.
# For x86 builds, change to "x86-windows-static".
set(VCPKG_TARGET_TRIPLET "x64-windows-static" CACHE STRING "")

# include(ExternalProject)
project(wxhelper VERSION 1.0.0)
enable_language(ASM_MASM)


# SET(CMAKE_ASM_NASM_FLAGS "-w0")


set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)

# Use static MSVC runtime (/MT) to match vcpkg x64-windows-static libraries
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")


file(GLOB CPP_FILES  ${PROJECT_SOURCE_DIR}/src/*.cc  ${PROJECT_SOURCE_DIR}/src/*.cpp ${PROJECT_SOURCE_DIR}/src/*.c )
file(GLOB ASM_FILES  ${PROJECT_SOURCE_DIR}/src/*.asm )
add_library(export_lib STATIC ${ASM_FILES})
set_target_properties(export_lib PROPERTIES LINKER_LANGUAGE C)


# add_subdirectory(spdlog)
add_subdirectory(source)

find_package(spdlog CONFIG REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)

find_path(DETOURS_INCLUDE_DIRS "detours/detours.h")
find_library(DETOURS_LIBRARY_RELEASE detours
             PATHS "${VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/lib"
             NO_DEFAULT_PATH
             REQUIRED)
find_library(DETOURS_LIBRARY_DEBUG detours
             PATHS "${VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/debug/lib"
             NO_DEFAULT_PATH
             REQUIRED)


add_library(wxhelper SHARED ${CPP_FILES} )


# Let imported targets provide their own include paths via target_link_libraries.
# Only detours needs manual include since it doesn't have a proper CMake config.
target_include_directories(wxhelper PRIVATE ${DETOURS_INCLUDE_DIRS})

target_link_libraries(wxhelper PRIVATE nlohmann_json::nlohmann_json)
target_link_libraries(wxhelper PRIVATE spdlog::spdlog)
target_link_libraries(wxhelper PRIVATE
                      optimized ${DETOURS_LIBRARY_RELEASE}
                      debug ${DETOURS_LIBRARY_DEBUG})
target_link_libraries(wxhelper PRIVATE export_lib)
target_compile_options(wxhelper PRIVATE /utf-8 /EHsc /std:c++17)


SET_TARGET_PROPERTIES(wxhelper PROPERTIES LINKER_LANGUAGE C
                      ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib
                      LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib
                      RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib
                      OUTPUT_NAME "wxhelper"
                      PREFIX "")
