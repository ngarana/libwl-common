# libwl-common — shared core for the qypr desktop suite.
#
# Consumers (waylaunch, qypr/lockscreen) graft this repo at
# third-party/libwl-common via git subtree and build selected units
# themselves; each repo keeps its own codegen rules, loop wiring and test
# harness. This build exists only for the units testable without a
# compositor (currently: render/BackdropBlur, cairo-only).
cmake_minimum_required(VERSION 3.20)
project(libwl-common CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(PkgConfig REQUIRED)
pkg_check_modules(CAIRO REQUIRED cairo)

option(BUILD_TESTING "Build the shared unit tests" ON)
if(BUILD_TESTING)
    enable_testing()
    add_executable(blur_test tests/blur_test.cpp render/BackdropBlur.cpp)
    target_include_directories(blur_test PRIVATE
        ${CMAKE_SOURCE_DIR}
        ${CAIRO_INCLUDE_DIRS})
    target_link_libraries(blur_test PRIVATE ${CAIRO_LIBRARIES})
    target_compile_options(blur_test PRIVATE ${CAIRO_CFLAGS} -Wall -Wextra -UNDEBUG)
    add_test(NAME blur_test COMMAND blur_test)
endif()
