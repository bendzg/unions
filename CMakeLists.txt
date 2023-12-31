cmake_minimum_required(VERSION 3.8)

project(unionfs-fuse VERSION 3.4
                     DESCRIPTION "user-space implementation of union filesystem (with help of fuse)"
                     LANGUAGES C)

INCLUDE(CheckIncludeFiles)

# Set a default build type for single-configuration
# CMake generators if no build type is set.
IF(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
	SET(CMAKE_BUILD_TYPE RelWithDebInfo)
ENDIF(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)

option(WITH_XATTR "Enable support for extended attributes" ON)

IF (WITH_XATTR)
	CHECK_INCLUDE_FILES("sys/xattr.h" HAVE_LIBC_XATTR)
	CHECK_INCLUDE_FILES("attr/xattr.h" HAVE_LIBATTR_XATTR)

	IF (HAVE_LIBC_XATTR)
		add_definitions(-DLIBC_XATTR)
	ELSEIF(HAVE_LIBATTR_XATTR)
		add_definitions(-DLIBATTR_XATTR)
	ENDIF()

	IF (NOT HAVE_LIBC_XATTR AND NOT HAVE_LIBATTR_XATTR)
		add_definitions(-DDISABLE_XATTR)
	ENDIF()
ELSE (WITH_XATTR)
	add_definitions(-DDISABLE_XATTR)
ENDIF (WITH_XATTR)

if (UNIX AND APPLE)
	option(WITH_LIBFUSE3 "Enable LibFUSE3 support" FALSE)  # supposedly macfuse (osxfuse?) still uses fuse2 api so let's make that a default for apple
	find_library(MACFUSE_PATH fuse HINTS /usr/local/lib)
	get_filename_component(MACFUSE_DIRECTORY ${MACFUSE_PATH} DIRECTORY)
	link_directories(${MACFUSE_DIRECTORY})
endif()

INSTALL(PROGRAMS mount.unionfs DESTINATION sbin)

add_subdirectory(src)
add_subdirectory(man)
