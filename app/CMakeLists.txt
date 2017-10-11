# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

set(PROJECT_NAME sqlite3)

set(LIB_STATIC ${PROJECT_NAME})
set(LIB_SHARED ${PROJECT_NAME}-shared)

# Change SQLITE_NEW_VERSION if you want to update the binary and record the SQLITE_NEW_VERSION content to SQLITE_CURRENT_VERSION
set(SQLITE_NEW_VERSION "3200100")
set(SQLITE_CURRENT_VERSION "3200100")
set(SQLITE_YEAR "2017")
set(SQLITE_DOWNLOAD_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../orgin")
set(SQLITE_LIBS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../sqlite3-libs")
set(SQLITE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src/main/c")

if(${SQLITE_NEW_VERSION} GREATER ${SQLITE_CURRENT_VERSION})
    set(SQLITE_BASENAME "sqlite-amalgamation-${SQLITE_NEW_VERSION}")
    set(SQLITE_URL "http://www.sqlite.org/${SQLITE_YEAR}/${SQLITE_BASENAME}.zip")

    if(EXISTS ${SQLITE_DOWNLOAD_DIR}/${SQLITE_BASENAME}.tar.gz)
    else()
        # Download the sqlite binary
        MESSAGE("Downloading ${SQLITE_URL}")
        file(DOWNLOAD ${SQLITE_URL} ${SQLITE_DOWNLOAD_DIR}/${SQLITE_BASENAME}.tar.gz STATUS status SHOW_PROGRESS)
        LIST(GET status 0 status_code)
        IF(NOT ${status_code} EQUAL 0)
            MESSAGE(FATAL_ERROR "Downloading sqlite file failed.\n ${status}\n")
        ENDIF(NOT ${status_code} EQUAL 0)

        # Extract the binary
        EXECUTE_PROCESS(COMMAND ${CMAKE_COMMAND} -E tar xzf ${SQLITE_DOWNLOAD_DIR}/${SQLITE_BASENAME}.tar.gz WORKING_DIRECTORY ${SQLITE_DOWNLOAD_DIR})
        configure_file(${SQLITE_DOWNLOAD_DIR}/${SQLITE_BASENAME}/sqlite3.c ${SQLITE_SOURCE_DIR} COPYONLY)
        configure_file(${SQLITE_DOWNLOAD_DIR}/${SQLITE_BASENAME}/sqlite3.h ${SQLITE_SOURCE_DIR} COPYONLY)
    endif()
endif()

# Compile the binary with recommended compile-times options, see https://sqlite.org/compile.html
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DSQLITE_DEFAULT_MEMSTATUS=0 -DSQLITE_OMIT_DEPRECATED -DSQLITE_OMIT_SHARED_CACHE")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DSQLITE_DEFAULT_MEMSTATUS=0 -DSQLITE_OMIT_DEPRECATED -DSQLITE_OMIT_SHARED_CACHE")

if (${DEBUG})
    set(COMPILE_MODE "debug")
else (${DEBUG})
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Os")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Os")
    set(COMPILE_MODE "release")
endif (${DEBUG})

MESSAGE( STATUS "CMAKE_C_FLAGS:   " ${CMAKE_C_FLAGS} )
MESSAGE( STATUS "CMAKE_CXX_FLAGS: " ${CMAKE_CXX_FLAGS} )

# Add static lib
file(GLOB SRC_FILES ${SQLITE_SOURCE_DIR}/*.c)
add_library(${LIB_STATIC} STATIC ${SRC_FILES} )
set_target_properties(${LIB_STATIC} PROPERTIES C_VISIBILITY_PRESET hidden)

# Add shared lib, this shared libary will be compiled and linked by app.apk and then extracted from the apk into dist directory. 
# The reason that we use the one extracted from apk is that the apk packager will help strip all the debug symbols of the .so 
# which will make all the .so be a smaller size.
add_library(${LIB_SHARED} SHARED ${SQLITE_SOURCE_DIR}/dummy/dummy.c )
target_link_libraries(${LIB_SHARED} -Wl,--whole-archive ${LIB_STATIC} -Wl,--no-whole-archive)
set_target_properties(${LIB_SHARED} PROPERTIES C_VISIBILITY_PRESET hidden)

set(SQLITE_DIST_DIR ${SQLITE_LIBS_DIR}/${COMPILE_MODE})

# ridirect the static lib binary to dist directory
set_target_properties(${LIB_STATIC}
                      PROPERTIES ARCHIVE_OUTPUT_DIRECTORY "${SQLITE_DIST_DIR}/libs/${ANDROID_ABI}")

