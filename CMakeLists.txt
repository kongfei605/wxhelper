cmake_minimum_required(VERSION 3.0.0)
# include(ExternalProject)
project(wxhelper VERSION 1.0.0)
enable_language(ASM_MASM)



# SET(CMAKE_ASM_NASM_FLAGS "-w0") 


set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)



set(CMAKE_CXX_STANDARD_REQUIRED ON)



file(GLOB CPP_FILES  ${PROJECT_SOURCE_DIR}/src/*.cc  ${PROJECT_SOURCE_DIR}/src/*.cpp ${PROJECT_SOURCE_DIR}/src/*.c )
file(GLOB ASM_FILES  ${PROJECT_SOURCE_DIR}/src/*.asm )
add_library(export_lib STATIC ${ASM_FILES})
set_target_properties(export_lib PROPERTIES LINKER_LANGUAGE C)

include_directories(${VCPKG_INSTALLED_DIR}/x64-windows/include ${DETOURS_INCLUDE_DIRS})
# include_directories(${VCPKG_INSTALLED_DIR}/x64-windows/include  ${PROJECT_SOURCE_DIR}/spdlog/include ) 



# add_subdirectory(spdlog)
add_subdirectory(source)

find_package(spdlog CONFIG REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)

find_path(DETOURS_INCLUDE_DIRS "detours/detours.h")
find_library(DETOURS_LIBRARY detours REQUIRED)


add_library(wxhelper SHARED ${CPP_FILES} )



target_link_libraries(wxhelper PRIVATE nlohmann_json::nlohmann_json)
target_link_libraries(wxhelper PRIVATE spdlog::spdlog spdlog::spdlog_header_only)
target_link_libraries(wxhelper PRIVATE ${DETOURS_LIBRARY})
target_link_libraries(wxhelper PRIVATE export_lib)
target_compile_options(wxhelper PRIVATE /utf-8 /EHsc /std:c++17)




SET_TARGET_PROPERTIES(wxhelper PROPERTIES LINKER_LANGUAGE C
                      ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib
                      LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib
                      RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib
                      OUTPUT_NAME "wxhelper"
                      PREFIX "")
           