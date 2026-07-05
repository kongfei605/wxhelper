cmake_minimum_required(VERSION 3.15)
cmake_policy(SET CMP0091 NEW)
project(injector VERSION 1.0.0)


set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Inherit static MSVC runtime from parent to stay consistent
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

file(GLOB INJECT_CPP_FILES  ${PROJECT_SOURCE_DIR}/*.cc  ${PROJECT_SOURCE_DIR}/*.cpp)

add_executable (injector  ${INJECT_CPP_FILES})

target_compile_definitions(injector PRIVATE _UNICODE UNICODE)

SET_TARGET_PROPERTIES(injector PROPERTIES LINKER_LANGUAGE C
                      ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin
                      LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin
                      RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin
                      OUTPUT_NAME "injector"
                      PREFIX "")