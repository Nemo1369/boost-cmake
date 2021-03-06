# Distributed under the Boost Software License, Version 1.0.
# See http://www.boost.org/LICENSE_1_0.txt

set(test_list
        basic
        # std
        # list_of
        ptr_list_of
        static_list_of
        tuple_list_of
        # list_inserter
        ptr_list_inserter
        ptr_map_inserter
        # list_of_workaround
        email_example
        my_vector_example
        multi_index_container
        )

set(test_targets)

if(BUILD_SHARED_LIBRARIES)
    set(Boost_USE_STATIC_LIBS FALSE)
    add_definitions(-DBOOST_TEST_DYN_LINK)
else()
    set(Boost_USE_STATIC_LIBS TRUE)
endif()

foreach(test IN LISTS test_list)
    set(target "${PROJECT_NAME}-test-${test}")
    add_executable(${target} ${test})
    target_link_libraries(${target} PRIVATE
            boost::assert
            boost::assign
            boost::circular_buffer
            boost::foreach
            boost::move
            boost::intrusive
            boost::multi_index
            boost::ptr_container
            boost::serialization
            boost::unit_test_framework
            )
    list(APPEND test_targets ${target})
    add_test(NAME ${PROJECT_NAME}.${test}
            COMMAND $<TARGET_FILE:${target}>
            )
endforeach()

add_custom_target(${PROJECT_NAME}-test
        COMMAND ${CMAKE_CTEST_COMMAND} -R "^${PROJECT_NAME}" ${CMAKE_BINARY_DIR}
        DEPENDS ${test_targets}
        )
