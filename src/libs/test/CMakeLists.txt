# Distributed under the Boost Software License, Version 1.0.
# See http://www.boost.org/LICENSE_1_0.txt

cmake_minimum_required(VERSION 2.8.11)
cmake_policy(SET CMP0042 NEW)

project(BoostTest CXX)

if(MSVC)
    # specify the asynchronous exception handling model
    string(REGEX REPLACE " ?/EH[as]?c?" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHa")
endif()

list(APPEND PRG_EXEC_MONITOR_SOURCES
        src/execution_monitor.cpp
        src/debug.cpp
        src/cpp_main.cpp
        )

add_definitions(-DBOOST_TEST_DYN_LINK=1)
add_definitions(-DBOOST_TEST_NO_AUTO_LINK=1)

if(BUILD_SHARED_LIBRARIES)
    add_library(boost_prg_exec_monitor SHARED
            ${PRG_EXEC_MONITOR_SOURCES}
            )
else()
    add_library(boost_prg_exec_monitor STATIC
            ${PRG_EXEC_MONITOR_SOURCES}
            )
endif()

if(BUILD_SHARED_LIBRARIES)
    add_definitions(-DBOOST_TEST_DYN_LINK=1)
    add_definitions(-DBOOST_TEST_NO_AUTO_LINK=1)
endif()

target_include_directories(boost_prg_exec_monitor PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
        )

target_link_libraries(boost_prg_exec_monitor PUBLIC
        boost::assert
        boost::algorithm
        boost::config
        boost::detail
        boost::concept_check
        boost::exception
        boost::function
        boost::type_traits
        boost::type_index
        )

if(BUILD_SHARED_LIBRARIES)
    add_library(boost_test_exec_monitor SHARED
            src/compiler_log_formatter.cpp
            src/debug.cpp
            src/decorator.cpp
            src/execution_monitor.cpp
            src/framework.cpp
            src/junit_log_formatter.cpp
            src/plain_report_formatter.cpp
            src/progress_monitor.cpp
            src/results_collector.cpp
            src/results_reporter.cpp
            src/test_tools.cpp
            src/test_tree.cpp
            src/unit_test_log.cpp
            src/unit_test_main.cpp
            src/unit_test_monitor.cpp
            src/unit_test_parameters.cpp
            src/xml_log_formatter.cpp
            src/xml_report_formatter.cpp
            )
else()
    add_library(boost_test_exec_monitor STATIC
            src/compiler_log_formatter.cpp
            src/debug.cpp
            src/decorator.cpp
            src/execution_monitor.cpp
            src/framework.cpp
            src/junit_log_formatter.cpp
            src/plain_report_formatter.cpp
            src/progress_monitor.cpp
            src/results_collector.cpp
            src/results_reporter.cpp
            src/test_tools.cpp
            src/test_tree.cpp
            src/unit_test_log.cpp
            src/unit_test_main.cpp
            src/unit_test_monitor.cpp
            src/unit_test_parameters.cpp
            src/xml_log_formatter.cpp
            src/xml_report_formatter.cpp
            )
endif()

target_include_directories(boost_test_exec_monitor PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
        )

target_link_libraries(boost_test_exec_monitor PRIVATE
        boost::assert
        boost::conversion
        boost::algorithm
        boost::align
        boost::math::special_functions
        boost::detail
        boost::concept_check
        boost::timer::header_only
        boost::function
        boost::type_traits
        boost::type_index
        boost::move
        boost::io
        boost::optional
        )

list(APPEND UNIT_TEST_FRAMEWORK_SOURCES
        src/compiler_log_formatter.cpp
        src/debug.cpp
        src/decorator.cpp
        src/execution_monitor.cpp
        src/framework.cpp
        src/junit_log_formatter.cpp
        src/plain_report_formatter.cpp
        src/progress_monitor.cpp
        src/results_collector.cpp
        src/results_reporter.cpp
        src/test_tools.cpp
        src/test_tree.cpp
        src/unit_test_log.cpp
        src/unit_test_main.cpp
        src/unit_test_monitor.cpp
        src/unit_test_parameters.cpp
        src/xml_log_formatter.cpp
        src/xml_report_formatter.cpp
        )

if(BUILD_SHARED_LIBRARIES)
    add_library(boost_unit_test_framework SHARED
            ${UNIT_TEST_FRAMEWORK_SOURCES}
            )
else()
    add_library(boost_unit_test_framework STATIC
            ${UNIT_TEST_FRAMEWORK_SOURCES}
            )
endif()

target_include_directories(boost_unit_test_framework PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        )

target_link_libraries(boost_unit_test_framework PUBLIC
        boost::align
        boost::conversion
        boost::concept_check
        boost::detail
        boost::math::special_functions
        boost::algorithm
        boost::timer::header_only
        boost::function
        boost::type_traits
        boost::type_index
        boost::iterator
        boost::move
        boost::io
        boost::optional
        boost::smart_ptr
        )

add_library(boost::prg_exec_monitor ALIAS boost_prg_exec_monitor)
set_property(TARGET boost_prg_exec_monitor PROPERTY EXPORT_NAME prg_exec_monitor)
add_library(boost::test_exec_monitor ALIAS boost_test_exec_monitor)
set_property(TARGET boost_test_exec_monitor PROPERTY EXPORT_NAME test_exec_monitor)
add_library(boost::unit_test_framework ALIAS boost_unit_test_framework)
set_property(TARGET boost_unit_test_framework PROPERTY EXPORT_NAME unit_test_framework)

install(DIRECTORY include/ DESTINATION include)

install(TARGETS
        boost_prg_exec_monitor
        boost_test_exec_monitor
        boost_unit_test_framework
        EXPORT ${PROJECT_NAME}Targets
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
        Algorithm
        Config
        Detail
        ConceptCheck
        Exception
        Function
        TypeTraits
        TypeIndex
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

export(TARGETS boost_unit_test_framework boost_prg_exec_monitor boost_test_exec_monitor
        NAMESPACE boost::
        FILE ${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Targets.cmake)

if(APPLE AND NOT ${CMAKE_TARGET_ARCHITECTURE} STREQUAL ${CMAKE_HOST_SYSTEM_PROCESSOR})
    set_target_properties(boost_unit_test_framework PROPERTIES XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY ${APPLE_CODE_SIGN_IDENTITY})
    set_target_properties(boost_unit_test_framework PROPERTIES XCODE_ATTRIBUTE_DEVELOPMENT_TEAM ${CMAKE_XCODE_ATTRIBUTE_DEVELOPMENT_TEAM})

    set_target_properties(boost_test_exec_monitor PROPERTIES XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY ${APPLE_CODE_SIGN_IDENTITY})
    set_target_properties(boost_test_exec_monitor PROPERTIES XCODE_ATTRIBUTE_DEVELOPMENT_TEAM ${CMAKE_XCODE_ATTRIBUTE_DEVELOPMENT_TEAM})

    set_target_properties(boost_prg_exec_monitor PROPERTIES XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY ${APPLE_CODE_SIGN_IDENTITY})
    set_target_properties(boost_prg_exec_monitor PROPERTIES XCODE_ATTRIBUTE_DEVELOPMENT_TEAM ${CMAKE_XCODE_ATTRIBUTE_DEVELOPMENT_TEAM})
endif()