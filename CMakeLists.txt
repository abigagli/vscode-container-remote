CMAKE_MINIMUM_REQUIRED(VERSION 3.14)
CMAKE_POLICY (VERSION 3.14)
CMAKE_POLICY (SET CMP0063 NEW)

#LIST(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/Modules)
PROJECT(REMOTEDEV LANGUAGES CXX VERSION 1.0)

INCLUDE (CheckIncludeFileCXX)
INCLUDE (GNUInstallDirs)
INCLUDE (GenerateExportHeader)
INCLUDE (CMakePackageConfigHelpers)



#See Effective Modern CMake, slides 31ff (https://github.com/boostcon/cppnow_presentations_2017/blob/master/05-19-2017_friday/effective_cmake__daniel_pfeifer__cppnow_05-19-2017.pdf)
#SET (BUILT_HERE ${PROJECT_NAME})
#MACRO (FIND_PACKAGE)
#    IF (NOT "${ARGV0}" IN_LIST BUILT_HERE)
#        _FIND_PACKAGE(${ARGV}) #Forwarding to the real find_package only if we've been asked to find something that's not built here
#    ENDIF()
#ENDMACRO()

OPTION (BUILD_SHARED_LIBS "" OFF)

#Undocumented options to disable in-source builds
SET(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
SET(CMAKE_DISABLE_SOURCE_CHANGES  ON)

IF("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_BINARY_DIR}")
    MESSAGE(SEND_ERROR "Do not build in-source. Please remove CMakeCache.txt and the CMakeFiles/ directory. Then build out-of-source.")
ENDIF()

FIND_PROGRAM(CCACHE_FOUND ccache)
IF(CCACHE_FOUND)
    #SET(ENV{CCACHE_PREFIX} "distcc")
    SET_PROPERTY(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
    SET_PROPERTY(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
ENDIF(CCACHE_FOUND)


SET (GENERATED_CODE_FOLDER ${CMAKE_BINARY_DIR}/generated_code)

IF (NOT CMAKE_BUILD_TYPE)
    MESSAGE (FATAL_ERROR "Please specify CMAKE_BUILD_TYPE")
ENDIF()


IF (NOT BUILD_SHARED_LIBS)
    SET (Boost_USE_STATIC_LIBS ON)
ENDIF()

SET (Boost_USE_MULTITHREADED ON)
FIND_PACKAGE (Boost 1.71 COMPONENTS system REQUIRED)


SET (CMAKE_CXX_STANDARD 17)
SET (CMAKE_CXX_STANDARD_REQUIRED ON)
SET (CMAKE_CXX_VISIBILITY_PRESET "hidden")
SET (CMAKE_C_VISIBILITY_PRESET "hidden")
SET (CMAKE_EXPORT_COMPILE_COMMANDS ON CACHE BOOL "Generate compile_commands.json for clang tools" FORCE)



#[==[
#COMMENTED OUT

GET_PROPERTY (KNOWN_FEATURES GLOBAL PROPERTY CMAKE_CXX_KNOWN_FEATURES)
MESSAGE ("Features known by CMAKE = ${KNOWN_FEATURES}")
MESSAGE ("Features supported by this compiler = ${CMAKE_CXX_COMPILE_FEATURES}")
#]==]


OPTION (BUILD_UNIT_TESTS "Build unit tests" ON)

IF (BUILD_UNIT_TESTS)
    ENABLE_TESTING()
    #INCLUDE(gmock)
ENDIF()


#Communicate with the codebase
SET (THIS_IS_ON ON)
SET (THIS_IS_OFF OFF)
SET (TEST1 CameFromTest1)
SET (TEST2 CameFromTest2)

#CHECK_INCLUDE_FILE_CXX ("systemd/sd-daemon.h" HAVE_SYSTEMD_DEV_PKG)
#CONFIGURE_FILE (compiletime_config/config.h.in ${GENERATED_CODE_FOLDER}/config_generated.h)

#Make sure everything is able to access generatd source files
INCLUDE_DIRECTORIES (${GENERATED_CODE_FOLDER})


MESSAGE ("\n")
MESSAGE ("Compiler ID: ${CMAKE_CXX_COMPILER_ID}")
MESSAGE ("Compiler VERSION: ${CMAKE_CXX_COMPILER_VERSION}")
MESSAGE ("System Name: ${CMAKE_SYSTEM_NAME}")
MESSAGE ("System Version: ${CMAKE_SYSTEM_VERSION}")
MESSAGE ("Build Type: ${CMAKE_BUILD_TYPE}")
MESSAGE ("Install Prefix: ${CMAKE_INSTALL_PREFIX}")
MESSAGE ("CMAKE_CXX_COMPILER: ${CMAKE_CXX_COMPILER}")
MESSAGE ("CMAKE_C_COMPILER: ${CMAKE_C_COMPILER}")
MESSAGE ("CMAKE_LINKER: ${CMAKE_LINKER}")
MESSAGE ("CMAKE_AR: ${CMAKE_AR}")
MESSAGE ("CMAKE_RANLIB: ${CMAKE_RANLIB}")
MESSAGE ("CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")
MESSAGE ("CMAKE_CXX_FLAGS_DEBUG: ${CMAKE_CXX_FLAGS_DEBUG}")
MESSAGE ("CMAKE_CXX_FLAGS_RELEASE: ${CMAKE_CXX_FLAGS_RELEASE}")
MESSAGE ("CMAKE_CXX_FLAGS_RELWITHDEBINFO: ${CMAKE_CXX_FLAGS_RELWITHDEBINFO}")
MESSAGE ("CMAKE_C_FLAGS: ${CMAKE_C_FLAGS}")
MESSAGE ("CMAKE_C_FLAGS_DEBUG: ${CMAKE_C_FLAGS_DEBUG}")
MESSAGE ("CMAKE_C_FLAGS_RELEASE: ${CMAKE_C_FLAGS_RELEASE}")
MESSAGE ("CMAKE_C_FLAGS_RELWITHDEBINFO: ${CMAKE_C_FLAGS_RELWITHDEBINFO}")
MESSAGE ("CMAKE_EXPORT_COMPILE_COMMANDS: ${CMAKE_EXPORT_COMPILE_COMMANDS}")
MESSAGE ("CMAKE_CROSSCOMPILING: ${CMAKE_CROSSCOMPILING}")
MESSAGE ("BOOST VERSION: ${Boost_VERSION}")
MESSAGE ("BOOST CONFIG: ${Boost_CONFIG}")
MESSAGE ("BOOST INCLUDES: ${Boost_INCLUDE_DIR}")

MESSAGE ("PATH: $ENV{PATH}")


# ###############################################
# configure RPATH handling so that it is correctly set both
# for build and install time
# (see http://www.cmake.org/Wiki/CMake_RPATH_handling and http://www.kitware.com/blog/home/post/510)

# use, i.e. don't skip the full RPATH for the build tree, this allows running
# executables from the build tree without having to mess with DYLD_LIBRARY_PATH etc..
SET(CMAKE_SKIP_BUILD_RPATH  FALSE)
#
# when building, don't use the install RPATH already
# (but later on when installing)
SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
#
# Do NOT add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
# This is to prevent rpaths pointing to external 3rdparties to remain into
# installed binaries (which should only rely on libraries found
# in the system and $ORIGIN/libexec)
# For example, say that a target links with /home/bla/3rdpartylib/lib/lib3rdparty.so
# during development it's ok for the target to have an rpath that contains /home/bla/3rdpartylib/lib
# but when installing, lib3rdparty.so will be copied into $ORIGIN/libexec and the installed target
# should not be able anymore to find it in /home/bla/3rpartylib/lib (but only in $ORIGIN/libexec)
# to avoid the possibility that on the deployed server it could pick up a different version
# of lib3rdparty.so if for some reason the deployed server has a /home/bla/3rdpartylib/lib/libparty.so too
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH FALSE)
#
#
# the RPATH to be used when installing, but only if it's not a system directory
LIST(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_INSTALL_PREFIX}/libexec" isSystemDir)
IF("${isSystemDir}" STREQUAL "-1")
    SET(CMAKE_INSTALL_RPATH "\$ORIGIN/libexec:\$ORIGIN/../libexec")
ENDIF()


FUNCTION(ECHO_ALL_CMAKE_VARIABLE_VALUES)
    MESSAGE(STATUS "")
    GET_CMAKE_PROPERTY(vs VARIABLES)
    FOREACH(v ${vs})
        MESSAGE(STATUS "${v}='${${v}}'")
    ENDFOREACH(v)
    MESSAGE(STATUS "")
ENDFUNCTION()



#SET (CODE_REVISION_SOURCE_FILE ${GENERATED_CODE_FOLDER}/code_revision.cpp)
#
##Set value for variable to be expanded in the following CONFIGURE_FILE command
#SET (CODE_REVISION_TEMPLATE_FILE ${CMAKE_SOURCE_DIR}/compiletime_config/code_revision.cpp.in)
#CONFIGURE_FILE (${CMAKE_SOURCE_DIR}/cmake/code_revision.cmake.in ${CMAKE_BINARY_DIR}/code_revision.cmake @ONLY)


############## TARGET-EXPORTING ################
#SET(EXPORT_TARGETS_NAME "${PROJECT_NAME}Targets")
#SET(VERSION_CONFIG "${GENERATED_CODE_FOLDER}/${PROJECT_NAME}ConfigVersion.cmake")
#SET(PROJECT_CONFIG "${GENERATED_CODE_FOLDER}/${PROJECT_NAME}Config.cmake")
#SET(CONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")
#SET(NAMESPACE "${PROJECT_NAME}::")
#
################### Config and Version ##################
##Generate a ${PROJECT_NAME}Version.cmake
#WRITE_BASIC_PACKAGE_VERSION_FILE (
#    "${VERSION_CONFIG}"
#    VERSION ${PROJECT_VERSION}
#    COMPATIBILITY SameMajorVersion
#    )
#
##Generate a ${PROJECT_NAME}Config.cmake to enable client projects to use FIND_PACKAGE
#CONFIGURE_PACKAGE_CONFIG_FILE (
#    "${PROJECT_SOURCE_DIR}/cmake/ProjectConfig.cmake.in"
#    "${PROJECT_CONFIG}"
#    #This must be the same as what is used after in INSTALL(FILES)
#    INSTALL_DESTINATION ${CONFIG_INSTALL_DIR}
#    )
#########################################################

##Define an export group to which each target that needs exporting can add itself.
#INSTALL (
#    EXPORT ${EXPORT_TARGETS_NAME}
#    NAMESPACE ${NAMESPACE}
#    DESTINATION ${CONFIG_INSTALL_DIR}
#    COMPONENT dev
#    )
#
##Copy the Config and Version files generated above into the proper install location
#INSTALL (
#    FILES
#        ${PROJECT_CONFIG}
#        ${VERSION_CONFIG}
#    #This must be the same as what is used above in CONFIGURE_PACKAGE_CONFIG_FILE
#    DESTINATION ${CONFIG_INSTALL_DIR}
#    )
#
##Export all targets in the target group "also" in the build directory, to be used by a consuming project (even if this project has not been installed) by including the *-buildtreeTargets.cmake file
#EXPORT (
#    EXPORT ${EXPORT_TARGETS_NAME}
#    NAMESPACE ${NAMESPACE}
#    FILE ${PROJECT_NAME}-buildtreeTargets.cmake #Consumers can just include this file
#    )
#

#ADD_SUBDIRECTORY (compiletime_config)
ADD_EXECUTABLE(remotedev main.cpp)


#For debug:
#ECHO_ALL_CMAKE_VARIABLE_VALUES()

MESSAGE ("\n")
SET(CPACK_PACKAGE_VENDOR "AB")
#SET(CPACK_GENERATOR "STGZ;TGZ")
SET(CPACK_GENERATOR "STGZ")
SET(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
SET(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
SET(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

INCLUDE (CPack)