cmake_minimum_required(VERSION 3.10.3)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_HOME_DIRECTORY}/Concrete/modules)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_HOME_DIRECTORY}/modules)

include(Concrete)

project(${CONCRETE_PROJECT_DEFAULT_PARAMETER})

concrete_project(
    netcode

    VERSION      1 2 1 0
    DESCRIPTION  "connection-oriented protocol built on top of UDP"
    HOMEPAGE_URL "https://github.com/networkprotocol/netcode.io"
    LANGUAGES    C CXX

    PACKAGE_NAME Netcode

    CONFIGURATION_TYPES Debug Release

    ROOT_DIR            ${CMAKE_HOME_DIRECTORY}/..
    BINARY_OUTPUT_DIR   ${CMAKE_HOME_DIRECTORY}/../bin
    LIBRARY_OUTPUT_DIR  ${CMAKE_HOME_DIRECTORY}/../lib
    WITH_COMPILER_TARGET_POSTFIX
)

concrete_set_global_properties(
    PROPERTIES 
        USE_FOLDERS ON 
        PREDEFINED_TARGETS_FOLDER "CMakeTargets"
)

set(SRC_DIR ${CONCRETE_PROJECT_ROOT_DIRECTORY})

if(MSVC)
    set(LIB_DIR ${SRC_DIR}/windows/lib/${CONCRETE_PROJECT_COMPILER_TARGET})
    
    concrete_target(    
        MSVC::Sodium

        IMPORTED
                
        TYPE 
            "Static" 

        IMPORTED_CONFIGURATIONS Debug Release
        
        INCLUDE_DIRECTORIES
            INTERFACE ${SRC_DIR}/windows ${SRC_DIR}/windows/sodium
            
        PROPERTIES
            IMPORTED_LOCATION_DEBUG 
                ${LIB_DIR}/Debug/${CONCRETE_GENERATOR_TOOLSET}/libsodium.lib

            IMPORTED_LOCATION_RELEASE
                ${LIB_DIR}/Release/${CONCRETE_GENERATOR_TOOLSET}/libsodium.lib
        )
endif()

concrete_global_target_configure(
    COMPILE_OPTIONS 
        "$<$<C_COMPILER_ID:MSVC>:-W4>"                      # warning leve 4
        "$<$<C_COMPILER_ID:MSVC>:-fp:fast>"                 # float point : fast
        "$<$<C_COMPILER_ID:MSVC>:-GR->"                     # RTTI OFF
        "$<$<C_COMPILER_ID:MSVC>:$<$<CONFIG:Debug>:-MTd>>"  # Mulit thread debug
        "$<$<C_COMPILER_ID:MSVC>:$<$<CONFIG:Release>:-MT>>" # Mulit thread

    LINK_LIBRARIES
        "$<$<C_COMPILER_ID:MSVC>:MSVC::Sodium>"
)

concrete_target( 
    netcode.io

    TYPE
        "Static"

    SOURCES
        ${SRC_DIR}/netcode.h
        ${SRC_DIR}/netcode.c

    FOLDER
        "Library" 
    
    INCLUDE_DIRECTORIES
        PUBLIC
            $<BUILD_INTERFACE:${SRC_DIR}>

        INTERFACE
            $<INSTALL_INTERFACE:include>
)


concrete_target(
    soak

    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/soak.c
    
    FOLDER
        "Example" 
    
    LINK_LIBRARIES
        PRIVATE netcode.io
)

concrete_target(
    client

    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/client.c
    
    FOLDER
        "Example" 

    LINK_LIBRARIES
        PRIVATE netcode.io
)

concrete_target(
    profile

    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/profile.c
    
    FOLDER
        "Example" 

    LINK_LIBRARIES
        PRIVATE netcode.io
)

concrete_target(
    server

    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/server.c
    
    FOLDER
        "Example" 

    LINK_LIBRARIES
        PRIVATE netcode.io
)

concrete_target(
    client_server
    
    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/client_server.c
    
    FOLDER
        "Example" 

    LINK_LIBRARIES
        PRIVATE netcode.io
)

concrete_target(
    cxx_test
    
    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/test.cpp
    
    FOLDER
        "Test"     
)

concrete_set_vs_startup_project(cxx_test)

# Install
set(NETCODE_INSTALL_INCLUDE "include" CACHE PATH "netcode.io library include path")
set(NETCODE_INSTALL_SODIUM "libsodium" CACHE PATH "sodium library path")
set(NETCODE_INSTALL_LIB "lib" CACHE PATH "netcode.io library files path")

concrete_install(DIRECTORY ${SRC_DIR}/windows DESTINATION ${NETCODE_INSTALL_SODIUM} FILES_MATCHING PATTERN "*.*" CONDITION "${MSVC}")
concrete_install(TARGETS netcode.io ARCHIVE DESTINATION ${NETCODE_INSTALL_LIB} DEFAULT_EXPORT)
concrete_install(FILES ${SRC_DIR}/netcode.h DESTINATION ${NETCODE_INSTALL_INCLUDE})

# Config
concrete_package_config(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in VERSION AnyNewerVersion DESTINATION "${NETCODE_INSTALL_LIB}/cmake")