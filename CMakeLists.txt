cmake_minimum_required (VERSION 3.5)
project (simplecpp LANGUAGES CXX)

#------------------------------------------------------------------------------
# Options
#------------------------------------------------------------------------------
option(SIMPLECPP_BUILD_APP "Should build preprocessor application?" ON)
option(SIMPLECPP_BUILD_SHARED "Should build library as DLL/.so ??" OFF)

set(SIMPLECPP_BUILD_TYPE STATIC)
if(${SIMPLECPP_BUILD_SHARED})
    set(SIMPLECPP_BUILD_TYPE SHARED)
endif()

#------------------------------------------------------------------------------
# Utils
#------------------------------------------------------------------------------
macro(custom_add_extra_flags targetArg)
    if (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
        target_compile_options(${targetArg} PRIVATE -Wall -Wextra -pedantic -Wcast-qual -Wfloat-equal -Wmissing-declarations -Wmissing-format-attribute -Wredundant-decls -Wshadow -Wundef)
    endif()

    if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        target_compile_options(${targetArg} PRIVATE -Weverything)
        # no need for c++98 compatibility
        target_compile_options(${targetArg} PRIVATE -Wno-c++98-compat-pedantic)
        # these are not really fixable
        target_compile_options(${targetArg} PRIVATE -Wno-exit-time-destructors -Wno-global-constructors)
        # TODO: fix these?
        target_compile_options(${targetArg} PRIVATE -Wno-zero-as-null-pointer-constant -Wno-padded -Wno-sign-conversion -Wno-conversion -Wno-old-style-cast)
    endif()
endmacro()


#------------------------------------------------------------------------------
# Definition of library
#------------------------------------------------------------------------------
add_library(simplecpp ${SIMPLECPP_BUILD_TYPE} simplecpp.cpp)
custom_add_extra_flags(simplecpp)
target_compile_options(simplecpp PRIVATE -std=c++03)
if (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    target_compile_options(simplecpp PRIVATE -Wno-long-long)
elseif (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(simplecpp PRIVATE -Wno-c++11-long-long)
endif()

#------------------------------------------------------------------------------
# Definition of app 
#------------------------------------------------------------------------------
if(${SIMPLECPP_BUILD_APP})
    add_executable(simplecpp-bin simplecpp.cpp main.cpp)
    custom_add_extra_flags(simplecpp-bin)
    set_property(TARGET simplecpp-bin PROPERTY CXX_STANDARD 11)
    target_link_libraries(simplecpp-bin simplecpp)
    set_target_properties(simplecpp-bin PROPERTIES OUTPUT_NAME simplecpp)
endif()

#------------------------------------------------------------------------------
# Definition of tests
#------------------------------------------------------------------------------
add_executable(testrunner simplecpp.cpp test.cpp)
set_property(TARGET testrunner PROPERTY CXX_STANDARD 11)

enable_testing()
add_test(NAME testrunner COMMAND testrunner)
add_test(NAME system-tests COMMAND python ${CMAKE_CURRENT_SOURCE_DIR}/run-tests.py)

#------------------------------------------------------------------------------
# Installation
#------------------------------------------------------------------------------
install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/simplecpp.h"
    DESTINATION include
)
install(TARGETS simplecpp 
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib/static
)
