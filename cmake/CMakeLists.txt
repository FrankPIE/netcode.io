cmake_minimum_required(VERSION 3.10.3)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_HOME_DIRECTORY}/Concrete/modules)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_HOME_DIRECTORY}/modules)

include(Concrete)

project(${CONCRETE_PROJECT_DEFAULT_PARAMETER})

concrete_project(
    NAME         netcode
    VERSION      1 2 1 0
    DESCRIPTION  "connection-oriented protocol built on top of UDP"
    HOMEPAGE_URL "https://github.com/networkprotocol/netcode.io"
    LANGUAGES    C CXX

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
    set(toolset ${CONCRETE_GENERATOR_TOOLSET})

    if ("${CONCRETE_GENERATOR_TOOLSET}" STREQUAL "msvc142")
        set(toolset "v142")
    endif()

    set(SODIUM_STATIC_DEBUG_LIB ${SRC_DIR}/windows/lib/${CONCRETE_PROJECT_COMPILER_TARGET}/Debug/${toolset}/libsodium.lib)
    set(SODIUM_STATIC_RELEASE_LIB ${SRC_DIR}/windows/lib/${CONCRETE_PROJECT_COMPILER_TARGET}/Release/${toolset}/libsodium.lib)

    concrete_target(    
        IMPORTED
        
        TARGET_NAME
            MSVC:Sodium
        
        TYPE 
            "Static"

        IMPORTED_CONFIGURATIONS Debug Release
        
        INCLUDE_DIRECTORIES
            INTERFACE ${SRC_DIR}/windows ${SRC_DIR}/windows/sodium      
            
        PROPERTIES
            IMPORTED_LOCATION_DEBUG ${SODIUM_STATIC_DEBUG_LIB}
            IMPORTED_LOCATION_RELEASE ${SODIUM_STATIC_RELEASE_LIB}
    )
    

endif()

concrete_global_target_configure(
    COMPILE_OPTIONS 
        "$<$<C_COMPILER_ID:MSVC>:-W4>"          # warning leve 4
        "$<$<C_COMPILER_ID:MSVC>:-fp:fast>"     # float point : fast
        "$<$<C_COMPILER_ID:MSVC>:-GR->"         # RTTI OFF
        "$<$<C_COMPILER_ID:MSVC>:$<$<CONFIG:Debug>:-MTd>>"  # Mulit thread debug
        "$<$<C_COMPILER_ID:MSVC>:$<$<CONFIG:Release>:-MT>>" # Mulit thread

    LINK_LIBRARIES
        "$<$<C_COMPILER_ID:MSVC>:MSVC:Sodium>"
)

concrete_target(
    TARGET_NAME
        netcode.io
    TYPE
        "Static"
    SOURCES
        ${SRC_DIR}/netcode.h
        ${SRC_DIR}/netcode.c
    FOLDER
        "Library" 
)

concrete_target(
    TARGET_NAME
        soak
    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/netcode.c
        ${SRC_DIR}/soak.c
    
    FOLDER
        "Example" 
)

concrete_target(
    TARGET_NAME
        client
    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/netcode.c
        ${SRC_DIR}/client.c
    
    FOLDER
        "Example" 
)

concrete_target(
    TARGET_NAME
        profile
    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/netcode.c
        ${SRC_DIR}/profile.c
    
    FOLDER
        "Example" 
)

concrete_target(
    TARGET_NAME
        server
    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/netcode.c
        ${SRC_DIR}/server.c
    
    FOLDER
        "Example" 
)

concrete_target(
    TARGET_NAME
        client_server
    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/netcode.c
        ${SRC_DIR}/client_server.c
    
    FOLDER
        "Example" 
)

concrete_target(
    TARGET_NAME
        cxx_test
    TYPE
        "Execute"
    SOURCES
        ${SRC_DIR}/test.cpp
    
    FOLDER
        "Test"     
)
