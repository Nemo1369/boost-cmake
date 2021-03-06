# Distributed under the Boost Software License, Version 1.0.
# See http://www.boost.org/LICENSE_1_0.txt

cmake_minimum_required(VERSION 2.8.11)
cmake_policy(SET CMP0042 NEW)

project(BoostGraphParallel CXX)

if(MSVC)
    add_definitions(
        /wd4251 # class needs to have dll-interface to be used by clients
        /wd4275 # non dll-interface class used as base for dll-interface class
        /wd4800 # forcing value to bool 'true' or 'false' (performance warning)
        )
endif()

list(APPEND CURRENT_TARGET_SOURCES
    src/mpi_process_group.cpp
    src/tag_allocator.cpp
    )

if(BUILD_SHARED_LIBRARIES)
    add_library(boost_graph_parallel SHARED
        ${CURRENT_TARGET_SOURCES}
        )
else()
    add_library(boost_graph_parallel STATIC
        ${CURRENT_TARGET_SOURCES}
        )
endif()

target_include_directories(boost_graph_parallel PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    )

target_link_libraries(boost_graph_parallel PUBLIC
    boost::assert
    boost::graph
    boost::mpi
    )

add_library(boost::graph::parallel ALIAS boost_graph_parallel)
set_property(TARGET boost_graph_parallel PROPERTY EXPORT_NAME graph_parallel)

install(DIRECTORY include/ DESTINATION include)

install(TARGETS boost_graph_parallel EXPORT ${PROJECT_NAME}Targets
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
    Graph
    MPI
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
    set_target_properties(boost_graph_parallel PROPERTIES XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY ${APPLE_CODE_SIGN_IDENTITY})
    set_target_properties(boost_graph_parallel PROPERTIES XCODE_ATTRIBUTE_DEVELOPMENT_TEAM ${CMAKE_XCODE_ATTRIBUTE_DEVELOPMENT_TEAM})
endif()