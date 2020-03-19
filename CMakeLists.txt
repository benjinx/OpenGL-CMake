CMAKE_MINIMUM_REQUIRED(VERSION 3.13 FATAL_ERROR)

# Initialize Projects
PROJECT(Demo VERSION 0.1)

# Options
OPTION(BUILD_THIRDPARTY "Build Missing ThirdParty Dependencies"             ON)

# Third Party Dependencies
IF(BUILD_THIRDPARTY)
    SET(THIRDPARTY_SOURCE_DIR ${CMAKE_SOURCE_DIR}/thirdparty)
    SET(THIRDPARTY_BINARY_DIR ${CMAKE_BINARY_DIR}/thirdparty)

    FILE(MAKE_DIRECTORY ${THIRDPARTY_BINARY_DIR})

    SET(glm_ROOT            ${THIRDPARTY_BINARY_DIR}/glm)
    SET(SDL2_ROOT           ${THIRDPARTY_BINARY_DIR}/SDL2)
    
    EXECUTE_PROCESS(
        COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" 
            -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
            -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
            ${THIRDPARTY_SOURCE_DIR}
            WORKING_DIRECTORY ${THIRDPARTY_BINARY_DIR}
        RESULT_VARIABLE _result
    )
    IF(_result)
        MESSAGE("${_result}")
        MESSAGE(FATAL_ERROR "Failed to configure thirdparty projects")
    ENDIF()

    SET(_THIRDPARTY_BUILD_COMMAND ${CMAKE_COMMAND} --build . )
    IF(CMAKE_GENERATOR STREQUAL "Unix Makefiles")
        SET(_THIRDPARTY_BUILD_COMMAND ${CMAKE_MAKE_PROGRAM} -l )
    ENDIF()

    EXECUTE_PROCESS(
        COMMAND ${_THIRDPARTY_BUILD_COMMAND}
        WORKING_DIRECTORY ${THIRDPARTY_BINARY_DIR}
        RESULT_VARIABLE _result
    )

    IF(_result)
        MESSAGE("${_result}")
        MESSAGE(FATAL_ERROR "Failed to build thirdparty projects")
    ENDIF()
ENDIF()

# Find Package
FIND_PACKAGE(glm CONFIG REQUIRED)
FIND_PACKAGE(SDL2 2.0.6 CONFIG REQUIRED)

# Runtime dir
GET_TARGET_PROPERTY(_location SDL2::SDL2 IMPORTED_LOCATION_RELEASE)
GET_FILENAME_COMPONENT(SDL2_RUNTIME_DIR ${_location} DIRECTORY)

SET(RUNTIME_SEARCH_PATH
    ${SDL2_RUNTIME_DIR}
)

# Glob
FILE(GLOB_RECURSE 
    _ENGINE_INCLUDES
    "${CMAKE_SOURCE_DIR}/include/*.hpp"
    "${CMAKE_SOURCE_DIR}/include/*.h"
)

FILE(GLOB_RECURSE 
    _ENGINE_SOURCES  
    "${CMAKE_SOURCE_DIR}/src/*.cpp"
    "${CMAKE_SOURCE_DIR}/src/*.c"
    "${CMAKE_SOURCE_DIR}/src/*.hpp"
    "${CMAKE_SOURCE_DIR}/src/*.h"
)

# Add source to executables
ADD_EXECUTABLE(
    ${PROJECT_NAME}
    ${_ENGINE_INCLUDES}
    ${_ENGINE_SOURCES}
)

# Runtime Search
IF(MSVC)
    SET_TARGET_PROPERTIES(
        ${PROJECT_NAME}
        PROPERTIES 
            VS_DEBUGGER_ENVIRONMENT "PATH=%PATH%;${RUNTIME_SEARCH_PATH}"
    )
ENDIF()


# Link
TARGET_LINK_LIBRARIES(
    ${PROJECT_NAME}
    PUBLIC
        glm
        SDL2::SDL2
)

IF(TARGET SDL2::SDL2main)
    TARGET_LINK_LIBRARIES(
        ${PROJECT_NAME}
        PUBLIC
            SDL2::SDL2main
    )
ENDIF()

# Make VS Happy
TARGET_COMPILE_OPTIONS(
    ${PROJECT_NAME}
    PUBLIC
        # Disable unknown pragmas warning, C++ exceptions
        $<$<CXX_COMPILER_ID:GNU>:-Wall -Wno-unknown-pragmas>
        $<$<CXX_COMPILER_ID:Clang>:-Wall -Wno-unknown-pragmas>
        $<$<CXX_COMPILER_ID:MSVC>:/MP /wd4068 /GR->
)

TARGET_COMPILE_DEFINITIONS(
    ${PROJECT_NAME}
    PUBLIC
        # Disable Visual Studio "not secure" warnings
        $<$<CXX_COMPILER_ID:MSVC>:_CRT_SECURE_NO_WARNINGS>
)

# Include Dirs
TARGET_INCLUDE_DIRECTORIES(
    ${PROJECT_NAME}
    PUBLIC
        $<INSTALL_INTERFACE:include>
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
        $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}/>
    PRIVATE
        "${CMAKE_SOURCE_DIR}/include"
        "${CMAKE_SOURCE_DIR}/include/imgui"
        "${CMAKE_SOURCE_DIR}/src"
        "${CMAKE_BINARY_DIR}/depend"
        "${CMAKE_BINARY_DIR}"
)

# Folders to filters
FOREACH(file IN ITEMS ${_ENGINE_INCLUDES} ${_ENGINE_SOURCES})
    FILE(RELATIVE_PATH file_path ${CMAKE_SOURCE_DIR} ${file})
    GET_FILENAME_COMPONENT(file_path ${file_path} DIRECTORY)
    FILE(TO_NATIVE_PATH ${file_path} file_path)
    SOURCE_GROUP(${file_path} FILES ${file})
ENDFOREACH()

# Set Startup project
SET_PROPERTY(DIRECTORY ${CMAKE_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME})
SET_TARGET_PROPERTIES(${PROJECT_NAME} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_BINARY_DIR}")