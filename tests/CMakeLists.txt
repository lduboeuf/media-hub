find_package(Threads)

add_definitions(-DBOOST_NO_CXX11_SCOPED_ENUMS)

include_directories(${CMAKE_SOURCE_DIR}/include)

option(
    MEDIA_HUB_ENABLE_DBUS_TEST_RUNNER 
    "Rely on dbus test runner to start a private session for testing purposes" 
    ON
)

if (MEDIA_HUB_ENABLE_DBUS_TEST_RUNNER)
    find_program(DBUS_TEST_RUNNER_EXECUTABLE dbus-test-runner)
    message(STATUS "Executing test suite under dbus-test-runner")
endif (MEDIA_HUB_ENABLE_DBUS_TEST_RUNNER)

# Build with system gmock and embedded gtest
if (EXISTS "/usr/src/googletest")
    # As of version 1.8.0
    set (GMOCK_SOURCE_DIR "/usr/src/googletest/googlemock" CACHE PATH "gmock source directory")
    set (GMOCK_INCLUDE_DIRS "${GMOCK_SOURCE_DIR}/include" CACHE PATH "gmock source include directory")
    set (GTEST_INCLUDE_DIRS "/usr/src/googletest/googletest/include" CACHE PATH "gtest source include directory")
else()
    set (GMOCK_SOURCE_DIR "/usr/src/gmock" CACHE PATH "gmock source directory")
    set (GMOCK_INCLUDE_DIRS "/usr/include/gmock/include" CACHE PATH "gmock source include directory")
    set (GTEST_INCLUDE_DIRS "${GMOCK_SOURCE_DIR}/gtest/include" CACHE PATH "gtest source include directory")
endif()


add_subdirectory(${GMOCK_SOURCE_DIR} "${CMAKE_CURRENT_BINARY_DIR}/gmock")

include_directories (
  .
  
  ${GMOCK_INCLUDE_DIR}
  ${GTEST_INCLUDE_DIR}
)

configure_file(
    test_data.cpp.in
    ${CMAKE_CURRENT_BINARY_DIR}/test_data.cpp
)

add_library(
    media-hub-test-framework

    ${CMAKE_CURRENT_BINARY_DIR}/test_data.cpp
)

target_link_libraries(
    media-hub-test-framework
    ${Boost_LIBRARIES}
)

set(
    CTEST_ENVIRONMENT
    "COM_UBUNTU_MEDIA_SERVICE_AUDIO_SINK_NAME=fakesink"
)

#add_subdirectory(acceptance-tests)
add_subdirectory(test-track-list)
add_subdirectory(unit-tests)
