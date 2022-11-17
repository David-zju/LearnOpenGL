cmake_minimum_required(VERSION 3.12)
set(CMAKE_CXX_STANDARD 17)
project(learnOpenGL)
# find package
find_package(glfw3 CONFIG REQUIRED)
find_package(glad CONFIG REQUIRED)

add_executable(main main.cpp)
target_link_libraries(main PRIVATE glfw)
target_link_libraries(main PRIVATE glad::glad)
