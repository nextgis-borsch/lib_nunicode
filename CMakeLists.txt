cmake_minimum_required (VERSION 2.8.12)
project(libnu C)


# some init settings
set(CMAKE_COLOR_MAKEFILE ON)
# set path to additional CMake modules
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/borsch-local ${CMAKE_SOURCE_DIR}/borsch ${CMAKE_MODULE_PATH})

include(GNUInstallDirs)

set(INSTALL_LIB_DIR ${CMAKE_INSTALL_LIBDIR} CACHE INTERNAL "Installation directory for libraries" FORCE)
set(INSTALL_INC_DIR ${CMAKE_INSTALL_INCLUDEDIR} CACHE INTERNAL "Installation directory for headers" FORCE)

include(util)
include(util-local)
check_version(PACKAGE_VERSION_MAJOR PACKAGE_VERSION_MINOR)
set(VERSION ${PACKAGE_VERSION_MAJOR}.${PACKAGE_VERSION_MINOR})
report_version(${PROJECT_NAME} ${VERSION})

# we do it here in order to reach library name in source subdirectories
if(BUILD_SHARED_LIBS)
    set(LIB_NAME ${PROJECT_NAME})
else()
    set(LIB_NAME ${PROJECT_NAME}static)
endif()


option(NU_BUILD_TESTS "Build tests, only with Debug config" OFF)
option(NU_BUILD_SAMPLES "Build samples" OFF)

set(NU_BUILD_OPTIONS "-DNU_WITH_EVERYTHING")
add_definitions(-DNU_BUILD_STATIC)

include(FindAnyProject)
#find_anyproject(SQLite3)

# backward-compatibility
if(CMAKE_BUILD_TYPE STREQUAL "RELEASE")
	set(CMAKE_BUILD_TYPE "Release")
endif()

if(NU_BUILD_TESTS)
	set(CMAKE_CONFIGURATION_TYPES "Debug") # restrict configurations to Debug
	set(CMAKE_BUILD_TYPE "Debug")
endif()

if(CMAKE_BUILD_TYPE MATCHES "GCOV")
	set(CMAKE_CONFIGURATION_TYPES "GCOV") # restrict configurations to GCOV
	set(NU_BUILD_TESTS 1)
elseif(CMAKE_BUILD_TYPE MATCHES "PROF")
	set(CMAKE_CONFIGURATION_TYPES "PROF") # restrict configurations to PROF
endif()

if(CMAKE_C_COMPILER_ID STREQUAL "GNU" OR CMAKE_C_COMPILER_ID STREQUAL "Clang")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -Werror -pedantic -std=c99")

	if(NOT WIN32)
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
	endif(NOT WIN32)

	if(CMAKE_BUILD_TYPE MATCHES "PROF")
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2 -g -ggdb")
	elseif(CMAKE_BUILD_TYPE MATCHES "GCOV")
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -ggdb -fprofile-arcs -ftest-coverage")
	endif()
endif()

if(CMAKE_C_COMPILER_ID STREQUAL "GNU")
	set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -fdata-sections -ffunction-sections")
	set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} -s -Wl,--gc-sections")
	set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS_RELEASE} -s -Wl,--gc-sections")
endif(CMAKE_C_COMPILER_ID STREQUAL "GNU")

add_subdirectory(libnu)
add_subdirectory(unicode.org)

export(TARGETS ${LIB_NAME} FILE ${LIB_NAME}-exports.cmake EXPORT_LINK_INTERFACE_LIBRARIES)

if(NU_BUILD_SAMPLES)
#	add_subdirectory(samples)
endif(NU_BUILD_SAMPLES)

if(NU_BUILD_TESTS)
#	add_subdirectory(tests)
endif(NU_BUILD_TESTS)

if(CMAKE_BUILD_TYPE MATCHES PROF)
#	add_subdirectory(tests/prof)
	if(SQLITE3_FOUND)
#		add_subdirectory(sqlite3/prof)
	endif(SQLITE3_FOUND)
endif(CMAKE_BUILD_TYPE MATCHES PROF)

if(SQLITE3_FOUND)
#	add_subdirectory(sqlite3)
endif(SQLITE3_FOUND)
