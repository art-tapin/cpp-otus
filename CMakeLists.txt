cmake_minimum_required(VERSION 3.10)

# 'set' creates an environmental variable 
# we can add vars from the outside (look gh-actions):
# - run: cmake . -DPATCH_VERSION=${{ github.run_number }}
# CACHE INTERNAL - no public access
set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VESRION 0.0.${PATCH_VERSION})

project(helloworld VERSION ${PROJECT_VESRION})

find_package(Boost COMPONENTS unit_test_framework REQUIRED)

# only for #defines
#
# version.h.in (@variable_syntax@):
# #pragma once
# #cmakedefine PROJECT_VERSION_PATCH @PROJECT_VERSION_PATCH@
# 
# will be transformed into:
# 
#  #pragma once
# 
# #define PROJECT_VERSION_PATCH 1
configure_file(version.h.in version.h)

# We are building three items
add_executable(helloworld_cli main.cpp)
add_library(helloworld lib.cpp)
add_executable(test_version test_version.cpp)

set_target_properties(helloworld_cli helloworld test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)
target_include_directories(helloworld
    PRIVATE "${CMAKE_BINARY_DIR}"
)

# Passing define:
# #ifdef BOOST_TEST_DYN_LINK
# ...
# #endif
set_target_properties(test_version PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
)

# Dependancies
target_link_libraries(helloworld_cli PRIVATE
    helloworld
)
target_link_libraries(test_version
    ${Boost_LIBRARIES}
    helloworld
)

# OS-specific compiler options
if (MSVC)
    target_compile_options(helloworld_cli PRIVATE
        /W4
    )
    target_compile_options(helloworld PRIVATE
        /W4
    )
    target_compile_options(test_version PRIVATE
        /W4
    )
else ()
    target_compile_options(helloworld_cli PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(helloworld PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(test_version PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
endif()


# What should be copied into the 'bin' to form a package? 
# Allows for --target package
install(TARGETS helloworld_cli RUNTIME DESTINATION bin)

# Magic: generator of the deb packages
set(CPACK_GENERATOR DEB)

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

set(CPACK_PACKAGE_CONTACT example@example.com)

include(CPack) # helper module

enable_testing()

# Register those binaries as tests bin : name
# Allows for --target test
add_test(test_version test_version)
