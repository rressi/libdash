cmake_minimum_required(VERSION 3.5)

set (CMAKE_CXX_STANDARD 11)

file(GLOB_RECURSE networkpart_source *.cpp)

add_executable(libdash_networkpart_test ${networkpart_source})
target_link_libraries(libdash_networkpart_test dash)