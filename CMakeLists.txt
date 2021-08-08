cmake_minimum_required (VERSION 3.4.11)
project (mpfr)

option(ENABLE_TESTS "Enable tests" ON)

add_definitions(-D_GMP_IEEE_FLOATS -DMPFR_HAVE_GMP_IMPL -DHAVE_CONFIG_H)

file(GLOB src_filenames src/*.c src/*.h)
add_library(mpfr ${src_filenames} "build.vs19/get_patches.c")
target_link_libraries(mpfr "${MPIR_DIRECTORY}/mpir.lib")

# configure_file(src/mparam_h.in inc/mparam.h)
configure_file(build.vs19/mparam.h inc/mparam.h)

include_directories("${MPIR_DIRECTORY}" "${CMAKE_CURRENT_BINARY_DIR}/inc")

if (MSVC_VERSION GREATER 1600)
    set_property(GLOBAL PROPERTY USE_FOLDERS ON)
endif()

set(excluded_tests "RRTest.c" "tgeneric.c" "tgeneric_ui.c")

if (ENABLE_TESTS)
    include(CTest)
    
    file(GLOB lib_test_filenames tests/lib/*.c)
    add_library(lib_test ${lib_test_filenames})
    target_include_directories(lib_test PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/src" "${CMAKE_CURRENT_SOURCE_DIR}/tests")
    set_target_properties(lib_test PROPERTIES FOLDER "tests" )
    
    file(GLOB test_filenames tests/*.c)
    foreach(filename ${test_filenames})
        
        # skip excluded tests
        set(skip 0)
        foreach(et ${excluded_tests})
            if ("${filename}" MATCHES "${et}$")
                set(skip 1)
            endif()
        endforeach()
        if (skip)
            continue()
        endif()
        
        get_filename_component(test_name ${filename} NAME_WE)
        add_executable("${test_name}" ${filename})
        set_target_properties("${test_name}" PROPERTIES FOLDER "tests" )
        target_link_libraries("${test_name}" lib_test mpfr)
        target_include_directories("${test_name}" PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/src")
        add_test(NAME "${test_name}" COMMAND "${test_name}")
    endforeach()
endif()
