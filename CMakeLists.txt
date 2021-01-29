cmake_minimum_required(VERSION "3.0.0")
project(bitcoin)

message("I'm /")

enable_testing()

add_subdirectory(doc/man)
add_subdirectory(src)
