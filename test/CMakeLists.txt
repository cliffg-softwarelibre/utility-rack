# Copyright 2018 by Cliff Green
#
# https://github.com/cliffg-softwarelibre/utility-rack
#
# Distributed under the Boost Software License, Version 1.0.
# (See accompanying file LICENSE.txt or copy at http://www.boost.org/LICENSE_1_0.txt)

cmake_minimum_required ( VERSION 3.8 )

project ( utility-rack-test VERSION 1.0 LANGUAGES CXX )

set ( test_source_dir "${CMAKE_SOURCE_DIR}/test" )
set ( test_main_src "${test_source_dir}/main_test.cpp" )
# set ( test_include_dir "${test_source_dir}/include" )
set ( tester_suffix "_runner" )
set ( main_test_lib_name "main_test_lib" )

set ( test_sources 
    "${test_source_dir}/queue/wait_queue_test.cpp"
    "${test_source_dir}/timer/periodic_timer_test.cpp"
    "${test_source_dir}/utility/erase_where_test.cpp"
    "${test_source_dir}/utility/make_byte_array_test.cpp"
    "${test_source_dir}/utility/repeat_test.cpp"
    "${test_source_dir}/utility/shared_buffer_test.cpp" )

set ( OPTIONS "" )
set ( DEFINITIONS "" )

set ( header_dirs
    "${include_source_dir}"
#    "${test_include_dir}"
    )

if (ENV{CATCH2_INCLUDE_DIR} STREQUAL "" )
    set ( catch2_include_dir "../Catch2/single_include" )
else ()
    set ( catch2_include_dir ENV{CATCH2_INCLUDE_DIR} )
endif()

if (ENV{BOOST_INCLUDE_DIR} STREQUAL "" )
    set ( boost_include_dir "../boost_1_68_0" )
else ()
    set ( boost_include_dir ENV{BOOST_INCLUDE_DIR} )
endif()

if (ENV{NETWORKING_TS_INCLUDE_DIR} STREQUAL "" )
    set ( networking_ts_include_dir "../networking-ts-impl/include" )
else ()
    set ( networking_ts_include_dir ENV{NETWORKING_TS_INCLUDE_DIR} )
endif()

if (ENV{RING_SPAN_LITE_INCLUDE_DIR} STREQUAL "" )
    set ( ring_span_lite_include_dir "../ring-span-lite/include" )
else ()
    set ( ring_span_lite_include_dir ENV{RING_SPAN_LITE_INCLUDE_DIR} )
endif()

function ( add_target_dependencies target )
    find_package ( Catch2 REQUIRED PATHS "../Catch2")
    if ( Catch2_FOUND )
        target_include_directories ( ${target} PRIVATE ${Catch2_INCLUDE_DIRS} )
    else ()
        target_include_directories ( ${target} PRIVATE ${catch2_include_dir} )
    endif ()
    find_package ( Boost )
    if ( Boost_FOUND )
        target_include_directories ( ${target} PRIVATE ${Boost_INCLUDE_DIRS} )
    else ()
        target_include_directories ( ${target} PRIVATE ${boost_include_dir} )
    endif ()
    find_package ( networking-ts-impl )
    if ( networking-ts-impl_FOUND )
        target_include_directories ( ${target} PRIVATE ${networking-ts-impl_INCLUDE_DIRS} )
    else ()
        target_include_directories ( ${target} PRIVATE ${networking_ts_include_dir} )
    endif ()
    find_package ( ring-span-lite )
    if ( ring-span-lite_FOUND )
        target_include_directories ( ${target} PRIVATE ${ring-span-lite_INCLUDE_DIRS} )
    else ()
        target_include_directories ( ${target} PRIVATE ${ring_span_lite_include_dir} )
    endif ()
endfunction()

function ( add_target_info target )
    target_compile_features    ( ${target} PRIVATE cxx_std_17 )
    target_compile_options     ( ${target} PRIVATE ${OPTIONS} )
    target_compile_definitions ( ${target} PRIVATE ${DEFINITIONS} )
    target_include_directories ( ${target} PRIVATE ${header_dirs} )
    add_target_dependencies    ( ${target} )
endfunction()

add_library ( ${main_test_lib_name} STATIC ${test_main_src} )
add_target_info ( ${main_test_lib_name} )

function ( make_exe target src )
    add_executable        ( ${target} ${src} )
    add_target_info       ( ${target} )
    target_link_libraries ( ${target} PRIVATE pthread )
    target_link_libraries ( ${target} PRIVATE ${main_test_lib_name} )
    message ( "Test executable to create: ${target}" )
    add_test ( NAME ${target}${tester_suffix} COMMAND ${target} )
endfunction()

enable_testing()

foreach ( test_src IN LISTS test_sources )
    get_filename_component ( targ ${test_src} NAME_WE )
    message ( "Calling make_exe for: ${targ}" )
    make_exe ( ${targ} ${test_src} )
endforeach()

# end of file

