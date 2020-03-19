CMAKE_MINIMUM_REQUIRED(VERSION 3.13 FATAL_ERROR)

# Project Name
PROJECT(thirdparty)

# Load Dependencies
INCLUDE(ExternalProject)

# glm

SET(glm_ROOT       ${CMAKE_BINARY_DIR}/glm)
SET(glm_SOURCE_DIR ${CMAKE_BINARY_DIR}/glm_src)

FIND_PACKAGE(glm CONFIG QUIET)

IF(NOT glm_FOUND)
    ExternalProject_Add(thirdparty-glm
        GIT_REPOSITORY      "https://github.com/g-truc/glm.git"
        GIT_TAG             "0.9.9.3"
        GIT_SHALLOW         TRUE
        SOURCE_DIR          ${glm_SOURCE_DIR}
        CMAKE_ARGS          -DCMAKE_INSTALL_PREFIX=${glm_ROOT}
                            -DGLM_TEST_ENABLE=OFF
        BUILD_COMMAND       ""
        UPDATE_COMMAND      ""
    )
ENDIF()

# SDL2

SET(SDL2_ROOT       ${CMAKE_BINARY_DIR}/SDL2)
SET(SDL2_SOURCE_DIR ${CMAKE_BINARY_DIR}/SDL2_src)
SET(SDL2_BINARY_DIR ${CMAKE_BINARY_DIR}/SDL2_bin)

FIND_PACKAGE(SDL2 2.0.6 CONFIG QUIET)

IF(NOT SDL2_FOUND)
    IF(MSVC)
        SET(SDL2_PATCH_COMMAND ${GIT_EXECUTABLE} apply --ignore-space-change --ignore-whitespace -v ${CMAKE_SOURCE_DIR}/SDL2-vcruntime.patch || ver > nul)
    ENDIF()
    ExternalProject_Add(
        thirdparty-SDL2
        HG_REPOSITORY       "http://hg.libsdl.org/SDL"
        HG_TAG              "default"
        SOURCE_DIR          ${SDL2_SOURCE_DIR}
        BINARY_DIR          ${SDL2_BINARY_DIR}
        CMAKE_ARGS          -DCMAKE_INSTALL_PREFIX=${SDL2_ROOT}
                            -DBUILD_SHARED_LIBS=ON
        BUILD_COMMAND       ""
        UPDATE_COMMAND      ""
        TEST_COMMAND        ""
        INSTALL_COMMAND     ${CMAKE_COMMAND} --build . --target install --config Release
        PATCH_COMMAND       ${SDL2_PATCH_COMMAND}
    )
ENDIF()