# Distributed under the Boost Software License, Version 1.0.
# See http://www.boost.org/LICENSE_1_0.txt

cmake_minimum_required(VERSION 2.8.11)
cmake_policy(SET CMP0042 NEW)

project(BoostIOStreams C CXX)

find_package(BZip2)
find_package(ZLIB)

add_definitions(-DBOOST_IOSTREAMS_USE_DEPRECATED)

if(MSVC)
    add_definitions(
        /wd4244 # conversion from 'boost::iostreams::stream_offset' to 'size_t'
        /wd4251 # class needs to have dll-interface to be used by clients
        /wd4800 # forcing value to bool 'true' or 'false'
        )
endif()

set(sources
    src/file_descriptor.cpp
    src/mapped_file.cpp
    )


set(link_libraries)

if(ZLIB_FOUND)
    include_directories(${ZLIB_INCLUDE_DIRS})
    list(APPEND sources src/zlib.cpp src/gzip.cpp)
    list(APPEND link_libraries ${ZLIB_LIBRARIES})
endif(ZLIB_FOUND)

if(BZIP2_FOUND)
    include_directories(${BZIP2_INCLUDE_DIR})
    list(APPEND sources src/bzip2.cpp)
    list(APPEND link_libraries ${BZIP2_LIBRARIES})
endif(BZIP2_FOUND)

if(BUILD_SHARED_LIBRARIES)
    add_library(boost_iostreams SHARED
        ${sources}
        )
else()
    add_library(boost_iostreams STATIC
        ${sources}
        )
endif()

target_include_directories(boost_iostreams PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    )

target_link_libraries(boost_iostreams PUBLIC
    ${link_libraries}
    boost::assert
    boost::config
    boost::range
    boost::detail
    boost::exception
    boost::type_traits
    boost::integer
    boost::smart_ptr
    )

add_library(boost::iostreams ALIAS boost_iostreams)
set_property(TARGET boost_iostreams PROPERTY EXPORT_NAME iostreams)

install(DIRECTORY include/ DESTINATION include)

install(TARGETS boost_iostreams EXPORT ${PROJECT_NAME}Targets
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    )

install(EXPORT ${PROJECT_NAME}Targets
    FILE ${PROJECT_NAME}Targets.cmake
    NAMESPACE boost::
    DESTINATION lib/cmake/${PROJECT_NAME}
    )

write_config_file("${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Config.cmake"
    Config
    Range
    Detail
    Exception
    TypeTraits
    Integer
    SmartPtr
    )
write_basic_package_version_file("${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${BOOST_VERSION}
    COMPATIBILITY AnyNewerVersion
    )

install(FILES
    "${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Config.cmake"
    "${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION lib/cmake/${PROJECT_NAME}
    )

export(TARGETS boost_iostreams
    NAMESPACE boost::
    FILE ${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Targets.cmake)

if(APPLE AND NOT ${CMAKE_TARGET_ARCHITECTURE} STREQUAL ${CMAKE_HOST_SYSTEM_PROCESSOR})
    set_target_properties(boost_iostreams PROPERTIES XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY ${APPLE_CODE_SIGN_IDENTITY})
    set_target_properties(boost_iostreams PROPERTIES XCODE_ATTRIBUTE_DEVELOPMENT_TEAM ${CMAKE_XCODE_ATTRIBUTE_DEVELOPMENT_TEAM})
endif()