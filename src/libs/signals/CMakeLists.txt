# Distributed under the Boost Software License, Version 1.0.
# See http://www.boost.org/LICENSE_1_0.txt

cmake_minimum_required(VERSION 2.8.11)
cmake_policy(SET CMP0042 NEW)

project(BoostSignals CXX)

if(MSVC)
    add_definitions(
        /wd4251 # class needs to have dll-interface to be used by clients
        /wd4275 # 'boost::noncopyable' used as base for dll-interface class
        )
endif()

list(APPEND CURRENT_TARGET_SOURCES
    src/trackable.cpp
    src/connection.cpp
    src/named_slot_map.cpp
    src/signal_base.cpp
    src/slot.cpp
    )

if(BUILD_SHARED_LIBRARIES)
    add_library(boost_signals SHARED
        ${CURRENT_TARGET_SOURCES}
        )
else()
    add_library(boost_signals STATIC
        ${CURRENT_TARGET_SOURCES}
        )
endif()

target_include_directories(boost_signals PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    )

target_link_libraries(boost_signals PUBLIC
    boost::assert
    boost::align
    boost::any
    boost::config
    boost::detail
    boost::exception
    boost::function
    boost::type_traits
    boost::type_index
    boost::iterator
    )

add_library(boost::signals ALIAS boost_signals)
set_property(TARGET boost_signals PROPERTY EXPORT_NAME signals)

install(DIRECTORY include/ DESTINATION include)

install(TARGETS boost_signals EXPORT ${PROJECT_NAME}Targets
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
        Any
        Config
        Detail
        Exception
        Function
        TypeTraits
        Iterator
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

export(TARGETS
        NAMESPACE boost::
        FILE ${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Targets.cmake)

if(APPLE AND NOT ${CMAKE_TARGET_ARCHITECTURE} STREQUAL ${CMAKE_HOST_SYSTEM_PROCESSOR})
    set_target_properties(boost_signals PROPERTIES XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY ${APPLE_CODE_SIGN_IDENTITY})
    set_target_properties(boost_signals PROPERTIES XCODE_ATTRIBUTE_DEVELOPMENT_TEAM ${CMAKE_XCODE_ATTRIBUTE_DEVELOPMENT_TEAM})
endif()