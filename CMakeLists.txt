cmake_minimum_required(VERSION 3.16)

project(libplist LANGUAGES C CXX)

#######################################################
###  cnary library - internal only
#######################################################

add_library(cnary STATIC)

target_sources(cnary
    PRIVATE
        libcnary/cnary.c
        libcnary/node_list.c
        libcnary/node.c
)

target_include_directories(cnary
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/libcnary/include>
        # $<INSTALL_INTERFACE:include/mylib>
)


#######################################################
###  plist C library
#######################################################

add_library(plist SHARED)

target_sources(plist
    PRIVATE
        src/base64.c
        src/base64.h
        src/bytearray.c
        src/bytearray.h
        src/strbuf.h
        src/hashtable.c
        src/hashtable.h
        src/ptrarray.c
        src/ptrarray.h
        src/time64.c
        src/time64.h
        src/time64_limits.h
        src/xplist.c
        src/bplist.c
        src/jsmn.c
        src/jsmn.h
        src/jplist.c
        src/plist.c
        src/plist.h
)

target_include_directories(plist
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include/>
    PRIVATE
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src>
)

target_compile_definitions(plist PRIVATE HAVE_STRNDUP)

target_link_libraries(plist PRIVATE cnary)

set_target_properties(plist PROPERTIES 
    VERSION 3.3.0
    SOVERSION 3 
    C_VISIBILITY_PRESET hidden)


#######################################################
###  plist C++ library
#######################################################

add_library(plist++ SHARED)

target_sources(plist++
    PRIVATE
        src/Node.cpp
        src/Structure.cpp
        src/Array.cpp
        src/Boolean.cpp
        src/Data.cpp
        src/Date.cpp
        src/Dictionary.cpp
        src/Integer.cpp
        src/Key.cpp
        src/Real.cpp
        src/String.cpp
        src/Uid.cpp 
    )

target_link_libraries(plist++ PUBLIC plist-2.0)

target_include_directories(plist++
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include/>
)

set_target_properties(plist++ PROPERTIES 
    VERSION 3.3.0
    SOVERSION 3)

#######################################################
###  install
#######################################################

install(TARGETS plist plist++
        EXPORT libplist-targets
        LIBRARY DESTINATION lib
)

install(EXPORT libplist-targets
        NAMESPACE plist-2.0
        FILE libplist-targets.cmake
        DESTINATION lib/cmake/libplist
)

# install(FILES MathFunctions.h DESTINATION include)

INSTALL (
    DIRECTORY ${CMAKE_SOURCE_DIR}/include/plist
    DESTINATION include
    FILES_MATCHING PATTERN "*.h*")


include(CMakePackageConfigHelpers)

# generate the config file that is includes the exports
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/libplist-config.cmake"
  INSTALL_DESTINATION "lib/cmake/libplist"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/libplist-config.cmake
    DESTINATION lib/cmake/libplist
)