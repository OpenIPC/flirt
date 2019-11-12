cmake_minimum_required(VERSION 3.0.2 FATAL_ERROR)

set(PROJECT_PRESENT_NAME "Flirt")
string(TOLOWER ${PROJECT_PRESENT_NAME} PROJECT)
project (${PROJECT} LANGUAGES C)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_FLAGS "-std=gnu99")

cmake_policy(SET CMP0069 NEW)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

if(CMAKE_BUILD_TYPE STREQUAL "Release")
    set(CMAKE_C_FLAGS
        "${CMAKE_C_FLAGS} -s -Os -ffunction-sections -Wl,--gc-sections -DNDEBUG -Wextra -Wno-builtin-macro-redefined")
    add_definitions(-DLOG_LEVEL=2)
else()
    if(BUILD_SHARED_LIBS)
        set(CMAKE_C_FLAGS
            "${CMAKE_C_FLAGS} -O1 -g -fsanitize=address -fno-omit-frame-pointer -Wno-builtin-macro-redefined")
    endif()
    add_definitions(-DLOG_LEVEL=0 -DDEBUG)
endif()

add_definitions(-DTHREAD_SAFE)

if(NOT SKIP_VERSION)
    set(VERSION_SRC ${CMAKE_CURRENT_BINARY_DIR}/version.c)
    # Add a custom command that produces version.c, plus a dummy output that's not
    # actually produced, in order to force version.cmake to always be re-run
    # before the build
    add_custom_command(
        OUTPUT ${VERSION_SRC} ${CMAKE_CURRENT_BINARY_DIR}/_version.c
        COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/version.cmake)
else()
    add_definitions(-DSKIP_VERSION)
endif()

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

add_definitions(
    -DBUILD_TYPE="${CMAKE_BUILD_TYPE}"
    -DPROJECT_NAME="${PROJECT_PRESENT_NAME}"
    -DPROJECT_FOLDER="${PROJECT}"
    -DLOG_USE_COLOR
    -DYAML
    -DVIDEO_FPS=25
)

include(FetchContent)

# # LIBEVENT OPTIONS #
# set (EVENT__LIBRARY_TYPE    STATIC CACHE INTERNAL
#     "Set library type to SHARED/STATIC/BOTH (default SHARED for MSVC, otherwise BOTH)") # SHARED/STATIC/BOTH
# set (EVENT__DISABLE_SAMPLES ON CACHE INTERNAL "Disable sample files")
# set (EVENT__DISABLE_BENCHMARK ON CACHE INTERNAL
#     "Defines if libevent should build without the benchmark executables")
# set (EVENT__DISABLE_TESTS   ON CACHE INTERNAL "If tests should be compiled or not")
# set (EVENT__DISABLE_REGRESS ON CACHE INTERNAL "Disable the regress tests")
# # set (EVENT__DISABLE_OPENSSL ON CACHE INTERNAL "Turn off OPENSSL")
# set (EVENT__DISABLE_OPENSSL OFF CACHE INTERNAL "Turn off OPENSSL")
# set (EVENT__DISABLE_MBEDTLS ON CACHE INTERNAL "Turn on MBEDTLS")

# # set (EVENT__ENABLE_VERBOSE_DEBUG ON CACHE INTERNAL "Enables verbose debugging")

# # LIBEVENT OPTIONS #

FetchContent_Declare(
    log.c
    URL https://github.com/SSharshunov/log.c/archive/refs/tags/latest.tar.gz
)

FetchContent_Declare(
    libhl
    URL https://github.com/xant/libhl/archive/refs/tags/libhl-3.1.tar.gz
)

FetchContent_Declare(
    pm
    GIT_REPOSITORY https://github.com/SSharshunov/pm.git
    GIT_TAG main
    # URL https://github.com/SSharshunov/pm/archive/refs/tags/latest.tar.gz
)

FetchContent_MakeAvailable(log.c libhl pm)

set (PROJECT_SRC
    main.c
    "${log.c_SOURCE_DIR}/src/log.c"
    "${pm_SOURCE_DIR}/src/pm.c"
    ${libhl_SOURCE_DIR}/src/linklist.c
    ${libhl_SOURCE_DIR}/src/hashtable.c
    ${VERSION_SRC}
)

add_executable(${PROJECT} ${PROJECT_SRC})
# target_compile_options(${PROJECT} PRIVATE -Wno-invalid-pch)

target_include_directories(${PROJECT}
    PUBLIC
    ${PROJECT_SOURCE_DIR}/src
    ${PROJECT_SOURCE_DIR}/src/plugins
    ${PROJECT_SOURCE_DIR}/src/utils
    PRIVATE
    "${log.c_SOURCE_DIR}/src"
    "${pm_SOURCE_DIR}/src"
    # "${libevent_BINARY_DIR}/include"
    # "${libevent_SOURCE_DIR}/include"
    # "${libyaml_SOURCE_DIR}/include"
    "${libhl_SOURCE_DIR}/src"
    # "${smolrtsp-libevent_SOURCE_DIR}/include"
    # "${smolrtsp_SOURCE_DIR}/include"
)

target_link_libraries(${PROJECT} PUBLIC
    yaml
    event
    stdc++ pthread gcc_s dl m c rt
)

set_target_properties(${PROJECT} PROPERTIES C_STANDARD 99 C_STANDARD_REQUIRED ON)

