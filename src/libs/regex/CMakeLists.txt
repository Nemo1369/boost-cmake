# Distributed under the Boost Software License, Version 1.0.
# See http://www.boost.org/LICENSE_1_0.txt

cmake_minimum_required(VERSION 2.8.11)
cmake_policy(SET CMP0042 NEW)

project(BoostRegex CXX)

#find_package(ICU)
#if(ICU_FOUND)
#    add_definitions(-DBOOST_HAS_ICU=1)
#    include_directories(${ICU_INCLUDE_DIR})
#endif()

list(APPEND CURRENT_TARGET_SOURCES
    src/c_regex_traits.cpp
        src/cpp_regex_traits.cpp
        src/cregex.cpp
        src/fileiter.cpp
        src/icu.cpp
        src/instances.cpp
        src/posix_api.cpp
        src/regex.cpp
        src/regex_debug.cpp
        src/regex_raw_buffer.cpp
        src/regex_traits_defaults.cpp
        src/static_mutex.cpp
        src/w32_regex_traits.cpp
        src/wc_regex_traits.cpp
        src/wide_posix_api.cpp
        src/winstances.cpp
        src/usinstances.cpp
    )

if(BUILD_SHARED_LIBRARIES)
    add_library(boost_regex SHARED
        ${CURRENT_TARGET_SOURCES}
        )
else()
    add_library(boost_regex STATIC
        ${CURRENT_TARGET_SOURCES}
        )
endif()

target_include_directories(boost_regex PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
        )

target_link_libraries(boost_regex PUBLIC
        ${ICU_LIBRARIES}
        ${ICU_I18N_LIBRARIES}
        boost::assert
        boost::config
        boost::detail
        boost::exception
        boost::type_traits
        boost::functional
        boost::integer
        boost::iterator
        boost::utility
        boost::smart_ptr
        )

add_library(boost::regex ALIAS boost_regex)
set_property(TARGET boost_regex PROPERTY EXPORT_NAME regex)

install(DIRECTORY include/ DESTINATION include)

install(TARGETS boost_regex EXPORT ${PROJECT_NAME}Targets
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
        Detail
        Exception
        TypeTraits
        Functional
        Integer
        Iterator
        Utility
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

export(TARGETS boost_regex
        NAMESPACE boost::
        FILE ${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Targets.cmake)

if(APPLE AND NOT ${CMAKE_TARGET_ARCHITECTURE} STREQUAL ${CMAKE_HOST_SYSTEM_PROCESSOR})
    set_target_properties(boost_regex PROPERTIES XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY ${APPLE_CODE_SIGN_IDENTITY})
    set_target_properties(boost_regex PROPERTIES XCODE_ATTRIBUTE_DEVELOPMENT_TEAM ${CMAKE_XCODE_ATTRIBUTE_DEVELOPMENT_TEAM})
endif()