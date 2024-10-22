cmake_minimum_required (VERSION 3.10)
cmake_policy(VERSION 3.10)

set(NAME VulkanRenderer)
project(${NAME})

set(CMAKE_CXX_STANDARD 17) # this does nothing for MSVC, use target_compile_options below
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

# Use FindVulkan module added with CMAKE 3.7
if(NOT CMAKE_VERSION VERSION_LESS 3.7.0)
    message(STATUS "Using module to find Vulkan")
    find_package(Vulkan)
endif()

if(NOT Vulkan_FOUND)
    message(FATAL_ERROR "Could not find Vulkan library!")
else()
    message(STATUS "Vulkan include dir: ${Vulkan_INCLUDE_DIRS}")
    include_directories(${Vulkan_INCLUDE_DIRS})
    message(STATUS "Vulkan library dir: ${Vulkan_LIBRARY}")
endif()

# vulkan lib
if(WIN32)
    # Nothing here (yet)
elseif(APPLE)
    link_libraries(${Vulkan_LIBRARY} "-framework AppKit" "-framework QuartzCore")
else(WIN32)
    link_libraries(${XCB_LIBRARIES} ${Vulkan_LIBRARY} ${Vulkan_LIBRARY} ${DIRECTFB_LIBRARIES} ${WAYLAND_CLIENT_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
endif(WIN32)

# cmake flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DVK_USE_PLATFORM_MACOS_MVK -DVK_EXAMPLE_XCODE_GENERATED")
# Set preprocessor defines
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DNOMINMAX -D_USE_MATH_DEFINES")

# Clang specific stuff
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-switch-enum")
endif()

# Compiler specific stuff
if(MSVC)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc")
elseif(APPLE)
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fobjc-arc -ObjC++")
    else()
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fobjc-arc -xobjective-c++")
    endif()
endif(MSVC)

# Third lib includes
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/lib/includes)
link_directories(${CMAKE_CURRENT_SOURCE_DIR}/lib/lib)

# Apple framework lib
if(APPLE)
    find_library(CORE_FOUNDATION_FRAMEWORK CoreFoundation)
    find_library(IOKIT_FRAMEWORK IOKit)
    find_library(COCOA_FRAMEWORK Cocoa)
endif(APPLE)

# Executable function
function(create_project_from_sources chapter demo)
#    file(GLOB SOURCE
#            "src/${chapter}/${demo}/*.h"
#            "src/${chapter}/${demo}/*.cpp"
#            "src/${chapter}/${demo}/*.vs"
#            "src/${chapter}/${demo}/*.fs"
#            "src/${chapter}/${demo}/*.tcs"
#            "src/${chapter}/${demo}/*.tes"
#            "src/${chapter}/${demo}/*.gs"
#            "src/${chapter}/${demo}/*.cs"
#    )
    # Executable
    file(GLOB SOURCE "src/empty_window/main.cpp")
    add_executable(empty_window ${SOURCE})
    target_link_libraries(empty_window ${CORE_FOUNDATION_FRAMEWORK} ${IOKIT_FRAMEWORK} ${COCOA_FRAMEWORK})
    target_link_libraries(empty_window glfw3)

    # output
    set_target_properties(empty_window PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${chapter}")
    set_target_properties(empty_window PROPERTIES RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_SOURCE_DIR}/bin/${chapter}")
    set_target_properties(empty_window PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_SOURCE_DIR}/bin/${chapter}")

#    if (demo STREQUAL "")
#        SET(replaced "")
#        string(REPLACE "/" "_" replaced ${chapter})
#        set(NAME ${replaced})
#    else()
#        set(NAME "${chapter}__${demo}")
#    endif()
#    add_executable(${NAME} ${SOURCE})
#    target_link_libraries(${NAME} ${LIBS})
#    if(MSVC)
#        target_compile_options(${NAME} PRIVATE /std:c++17 /MP)
#        target_link_options(${NAME} PUBLIC /ignore:4099)
#    endif(MSVC)
#    if(WIN32)
#        set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${chapter}")
#        set_target_properties(${NAME} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${chapter}/Debug")
#    elseif(UNIX AND NOT APPLE)
#        set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${chapter}")
#    elseif(APPLE)
#        set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${chapter}")
#        set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_SOURCE_DIR}/bin/${chapter}")
#        set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_SOURCE_DIR}/bin/${chapter}")
#    endif(WIN32)
    # copy shader files to build directory
    file(GLOB SHADERS
            "src/${chapter}/${demo}/*.vs"
            # "src/${chapter}/${demo}/*.frag"
            "src/${chapter}/${demo}/*.fs"
            "src/${chapter}/${demo}/*.tcs"
            "src/${chapter}/${demo}/*.tes"
            "src/${chapter}/${demo}/*.gs"
            "src/${chapter}/${demo}/*.cs"
    )
    # copy dlls
#    file(GLOB DLLS "dlls/*.dll")
#    foreach(SHADER ${SHADERS})
#        if(WIN32)
#            # configure_file(${SHADER} "test")
#            add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${SHADER} $<TARGET_FILE_DIR:${NAME}>)
#            add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${DLLS} $<TARGET_FILE_DIR:${NAME}>)
#        elseif(UNIX AND NOT APPLE)
#            file(COPY ${SHADER} DESTINATION ${CMAKE_SOURCE_DIR}/bin/${chapter})
#        elseif(APPLE)
#            # create symbolic link for *.vs *.fs *.gs
#            get_filename_component(SHADERNAME ${SHADER} NAME)
#            makeLink(${SHADER} ${CMAKE_SOURCE_DIR}/bin/${chapter}/${SHADERNAME} ${NAME})
#        endif(WIN32)
#    endforeach(SHADER)
#    # if compiling for visual studio, also use configure file for each project (specifically to set up working directory)
#    if(MSVC)
#        configure_file(${CMAKE_SOURCE_DIR}/configuration/visualstudio.vcxproj.user.in ${CMAKE_CURRENT_BINARY_DIR}/${NAME}.vcxproj.user @ONLY)
#    endif(MSVC)
endfunction()

create_project_from_sources(empty_window empty_window)
