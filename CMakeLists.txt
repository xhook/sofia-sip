# the name of your project
project(sofia-sip-ua)
cmake_minimum_required(VERSION 2.8)

include(ExternalProject)

include(cmake/PlatformSettings.cmake)
include(cmake/ConfigureArtifactPath.cmake)
include(cmake/CommonCompilerFlags.cmake)

#message(STATUS ${OPENSSL_ROOT_DIR})
#find_package(OpenSSL)
#if (NOT OPENSSL_FOUND)
#    message(FATAL_ERROR "Need OpenSSL to build soifa-sip-ua")
#endif()

if (UNIX)

    set(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
    set(BINARY_DIR ${CMAKE_CURRENT_SOURCE_DIR})

    message(STATUS CMAKE_C_FLAGS=${CMAKE_C_FLAGS})
    message(STATUS CMAKE_MODULE_LINKER_FLAGS=${CMAKE_MODULE_LINKER_FLAGS})

    # Reomve our strict compilation rules
    string(REGEX REPLACE "-Werror" "" SOFIA_SIP_CFLAGS ${CMAKE_C_FLAGS})
    string(REGEX REPLACE "-Wall" "" SOFIA_SIP_CFLAGS ${SOFIA_SIP_CFLAGS})
    string(REGEX REPLACE "-Wextra" "" SOFIA_SIP_CFLAGS ${SOFIA_SIP_CFLAGS})

    message(STATUS SOFIA_SIP_CFLAGS=${SOFIA_SIP_CFLAGS})

    if (${PLATFORM} STREQUAL "Android")
        set(SOFIA_SIP_TARGET_PLATFORM "arm-android-linux")
        foreach(includeDir ${ANDROID_SYSTEM_INCLUDE_DIRS})
            set(SOFIA_SIP_ANDROID_INCLUDES "${SOFIA_SIP_ANDROID_INCLUDES} -I${includeDir}")
        endforeach()
        set(SOFIA_SIP_CFLAGS "${SOFIA_SIP_CFLAGS} ${SOFIA_SIP_ANDROID_INCLUDES}")
        foreach(libraryDir ${ANDROID_SYSTEM_LIB_DIRS})
            set(SOFIA_SIP_ANDROID_LIB_DIRS "${SOFIA_SIP_ANDROID_LIB_DIRS} -L${libraryDir}")
        endforeach()
        set(SOFIA_SIP_LDFLAGS "${SOFIA_SIP_LDFLAGS} ${SOFIA_SIP_ANDROID_LIB_DIRS}")
    elseif(${PLATFORM} STREQUAL "Linux")
        set(SOFIA_SIP_TARGET_PLATFORM "")
    elseif(${PLATFORM} STREQUAL "iOS")
        set(SOFIA_SIP_TARGET_PLATFORM "arm-apple-darwin10")
    elseif(${PLATFORM} STREQUAL "Mac OS X")
        set(SOFIA_SIP_TARGET_PLATFORM "")
    endif()

    message(STATUS SOFIA_SIP_CFLAGS=${SOFIA_SIP_CFLAGS})

    set(SOFIA_SIP_CFLAGS "${SOFIA_SIP_CFLAGS} -I${OPENSSL_INCLUDE_DIR}")
    set(SOFIA_SIP_LDFLAGS "${SOFIA_SIP_LDFLAGS} -l${OPENSSL_LIBRARIES}")

    message(STATUS SOFIA_SIP_CFLAGS=${SOFIA_SIP_CFLAGS})
    message(STATUS SOFIA_SIP_LDFLAGS=${SOFIA_SIP_LDFLAGS})

    configure_file(configure_command.sh.in configure_command.sh)

    set(CONFIGURE_COMMAND bash ${CMAKE_CURRENT_BINARY_DIR}/configure_command.sh)
    set(BUILD_COMMAND make clean && make -j4 && make install)

    message(STATUS "SOURCE_DIR = ${SOURCE_DIR}")
    message(STATUS "BINARY_DIR = ${BINARY_DIR}")
    message(STATUS "CONFIGURE_COMMAND = ${CONFIGURE_COMMAND}")
    message(STATUS "BUILD_COMMAND = ${BUILD_COMMAND}")

    ExternalProject_Add(sofia-sip-ua
        SOURCE_DIR ${SOURCE_DIR}
        BINARY_DIR ${BINARY_DIR}
        CONFIGURE_COMMAND ${CONFIGURE_COMMAND}
        BUILD_COMMAND ${BUILD_COMMAND})

elseif(${CMAKE_GENERATOR} STREQUAL "Visual Studio 2010")

    message(FATAL_ERROR "Fix Visual Studio build")

else()
 
    message(FATAL_ERROR "Generator/platform is not supported")

endif()
