# Distributed under the Boost Software License, Version 1.0.
# See http://www.boost.org/LICENSE_1_0.txt

cmake_minimum_required(VERSION 2.8.11)
cmake_policy(SET CMP0042 NEW)

project(BoostAtomic CXX)
if(BUILD_SHARED_LIBRARIES)
    add_library(boost_atomic SHARED
        src/lockpool.cpp
        )
else()
    add_library(boost_atomic STATIC
        src/lockpool.cpp
        )
endif()

target_include_directories(boost_atomic PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    )

target_link_libraries(boost_atomic PUBLIC
    boost::assert
    boost::config
    boost::detail
    boost::type_traits
    boost::integer
    boost::smart_ptr
    )

add_library(boost::atomic ALIAS boost_atomic)
set_property(TARGET boost_atomic PROPERTY EXPORT_NAME atomic)

install(DIRECTORY include/ DESTINATION include)

install(TARGETS boost_atomic EXPORT ${PROJECT_NAME}Targets
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

export(TARGETS boost_atomic
        NAMESPACE boost::
        FILE ${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Targets.cmake)

if(APPLE AND NOT ${CMAKE_TARGET_ARCHITECTURE} STREQUAL ${CMAKE_HOST_SYSTEM_PROCESSOR})
    set_target_properties(boost_atomic PROPERTIES XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY ${APPLE_CODE_SIGN_IDENTITY})
    set_target_properties(boost_atomic PROPERTIES XCODE_ATTRIBUTE_DEVELOPMENT_TEAM ${CMAKE_XCODE_ATTRIBUTE_DEVELOPMENT_TEAM})
endif()