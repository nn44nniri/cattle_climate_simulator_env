cmake_minimum_required(VERSION 3.16)
project(cattle_climate_cpp VERSION 0.3.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include(CTest)

enable_testing()

add_library(cattle_climate
    src/formulas.cpp
    src/settings.cpp
    src/simulator.cpp
    src/env.cpp
)

target_include_directories(cattle_climate
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/include
)

add_executable(cattle_climate_example src/main.cpp)
target_link_libraries(cattle_climate_example PRIVATE cattle_climate)

add_executable(cattle_climate_random_env_test tests/random_env_test.cpp)
target_link_libraries(cattle_climate_random_env_test PRIVATE cattle_climate)

add_test(
    NAME cattle_climate_random_env_test
    COMMAND cattle_climate_random_env_test ${CMAKE_CURRENT_SOURCE_DIR}/config/settings.json
)
