cmake_minimum_required(VERSION 3.20)

project(
  flappy-bird
  LANGUAGES C CXX
  DESCRIPTION "A flappy bird clone in C using `raylib`"
  VERSION 0.1.1
)

# Generate compile_commands.json
if("${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
  message(STATUS "Exporting compile_commands.json")
  set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
endif()

# Dependencies
set(RAYLIB_VERSION 5.0)
find_package(raylib ${RAYLIB_VERSION} QUIET) # QUIET or REQUIRED
if (NOT raylib_FOUND) # If there's none, fetch and build raylib
  include(FetchContent)
  FetchContent_Declare(
    raylib
    DOWNLOAD_EXTRACT_TIMESTAMP OFF
    URL https://github.com/raysan5/raylib/archive/refs/tags/${RAYLIB_VERSION}.tar.gz
  )
  FetchContent_GetProperties(raylib)
  if (NOT raylib_POPULATED) # Have we downloaded raylib yet?
    set(FETCHCONTENT_QUIET NO)
    FetchContent_Populate(raylib)
    set(BUILD_EXAMPLES OFF CACHE BOOL "" FORCE) # don't build the supplied examples
    add_subdirectory(${raylib_SOURCE_DIR} ${raylib_BINARY_DIR})
  endif()
endif()

# Our Project
# set project C standard (if applicable)
set(CMAKE_C_EXTENSIONS OFF) # turn off compiler extensions
set(CMAKE_C_STANDARD 11) # C11
set(CMAKE_C_STANDARD_REQUIRED)

add_executable(${PROJECT_NAME})
add_subdirectory(src)

set_target_properties(${PROJECT_NAME} PROPERTIES
  RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${PROJECT_NAME})

add_custom_command(
  TARGET ${PROJECT_NAME} POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/resources $<TARGET_FILE_DIR:${PROJECT_NAME}>/resources
  DEPENDS ${PROJECT_NAME})

#set(raylib_VERBOSE 1)
target_link_libraries(${PROJECT_NAME} raylib)

if(CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND MSVC)
  target_compile_options(
    ${PROJECT_NAME}
    PRIVATE 
    /clang:-Wall
    -Wextra
    $<$<CONFIG:Debug>:-Werror>)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL MSVC)
  target_compile_options(${PROJECT_NAME}
                         PRIVATE /MP /EHsc $<$<CONFIG:Debug>:/WX>)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL Clang OR CMAKE_CXX_COMPILER_ID STREQUAL GNU)
  target_compile_options(
    ${PROJECT_NAME}
    PRIVATE 
    -Wall
    -Wextra
    $<$<CONFIG:Debug>:-Werror>)
endif()

# Web Configurations
if (${PLATFORM} STREQUAL "Web")
  # Tell Emscripten to build an example.html file.
  set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".html")
  target_link_options(${PROJECT_NAME} PUBLIC "-sUSE_GLFW=3")
endif()