cmake_minimum_required(VERSION 3.9.2)

if(DEFINED $ENV{TRAVIS_BUILD_NUMBER})
  set(EX_VERSION 0.0.$ENV{TRAVIS_BUILD_NUMBER})
else()
  set(EX_VERSION 0.0.1)
endif()

project(helloworld VERSION ${EX_VERSION})

message(STATUS "Source directory: ${CMAKE_CURRENT_SOURCE_DIR}")
message(STATUS "Project version: ${PROJECT_VERSION}")

# preconfig

find_package(Boost COMPONENTS unit_test_framework REQUIRED)

message(STATUS "Project version patch: ${PROJECT_VERSION_PATCH}")
configure_file(version.h.in version.h)
configure_file(version.h.in ${CMAKE_CURRENT_SOURCE_DIR}/version.h @ONLY)

# binaries

add_executable(helloworld_cli "main.cpp")

add_library(helloworld lib.cpp)

add_executable(test_version test_version.cpp)

# properties

set_target_properties(helloworld_cli helloworld test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
    COMPILE_OPTIONS "-Wpedantic;-Wall;-Wextra"
)

set_target_properties(test_version PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
)

# linking

target_link_libraries(helloworld_cli
    helloworld
)

target_link_libraries(test_version
    ${Boost_LIBRARIES}
    helloworld
)

# package

install(TARGETS helloworld_cli RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

set(CPACK_PACKAGE_CONTACT borodin.devel@gmail.com)

include(CPack)

# tests

enable_testing()

add_test(test_version test_version)
