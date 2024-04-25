# XCPlite
cmake_minimum_required(VERSION 3.5)

set(CMAKE_C_COMPILER "gcc")
set(CMAKE_CXX_COMPILER "g++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

project(XCPlite VERSION 6.0 LANGUAGES C CXX)

set(XCPlite_SOURCES main.c src/xcpAppl.c src/xcpLite.c src/xcpEthTl.c src/xcpEthServer.c src/A2L.c src/platform.c )

add_executable(XCPlite ${XCPlite_SOURCES})
target_include_directories(XCPlite PUBLIC "${PROJECT_SOURCE_DIR}" )
target_include_directories(XCPlite PUBLIC "${PROJECT_SOURCE_DIR}/src" )
set_target_properties(XCPlite PROPERTIES SUFFIX ".out")
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
target_link_libraries(XCPlite PRIVATE Threads::Threads)
target_link_libraries(XCPlite PRIVATE m)
