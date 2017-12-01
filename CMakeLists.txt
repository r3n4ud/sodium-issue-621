project(dummy C)

## Add ./cmake to CMAKE_MODULE_PATH so we can resolve find_package(sodium) to
## Findsodium.cmake
set(CMAKE_MODULE_PATH
  ${PROJECT_SOURCE_DIR}/cmake
  ${CMAKE_MODULE_PATH})

#set(sodium_USE_STATIC_LIBS ON)
find_package(sodium REQUIRED)

add_executable(dummy main.c)
target_link_libraries(dummy general sodium)
