# Distributed under the Boost Software License, Version 1.0.
# See http://www.boost.org/LICENSE_1_0.txt

include(Ryppl)
include(RypplTestSuite)

ryppl_find_and_use_package(BoostDetail)
ryppl_find_and_use_package(BoostAny)
ryppl_find_and_use_package(BoostTest)

if(MSVC)
    # <asynch-exceptions>on
endif(MSVC)

ryppl_test_suite(
        RUN
        algorithm_test.cpp
        bind_tests_simple.cpp
        bind_tests_advanced.cpp
        bind_tests_simple_f_refs.cpp
        bll_and_function.cpp
        cast_test.cpp
        constructor_tests.cpp
        control_structures.cpp
        exception_test.cpp
        extending_rt_traits.cpp
        is_instance_of_test.cpp
        member_pointer_test.cpp
        operator_tests_simple.cpp
        phoenix_control_structures.cpp
        switch_construct.cpp
        result_of_tests.cpp
        ret_test.cpp
        NO_SINGLE_TARGET
)
