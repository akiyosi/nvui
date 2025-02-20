cmake_minimum_required(VERSION 3.19)
#if(DEFINED ENV{QT_DIR})
  #message("Qt Dir is Set")
  #list(APPEND CMAKE_PREFIX_PATH "$ENV{QT_DIR}")
  #list(APPEND CMAKE_PREFIX_PATH "$ENV{QT_DIR}/Qt5Core")
  #list(APPEND CMAKE_PREFIX_PATH "$ENV{QT_DIR}/Qt5Widgets")
  #list(APPEND CMAKE_PREFIX_PATH "$ENV{QT_DIR}/Qt5Gui")
#else()
  #message("Could not find QT_DIR")
#endif()
#if (DEFINED ENV{QT_SVGDIR})
  #message("Qt SVG Dir is set")
  #list(APPEND CMAKE_PREFIX_PATH "$ENV{QT_SVGDIR}")
  #list(APPEND CMAKE_PREFIX_PATH "$ENV{QT_SVGDIR}/Qt5Svg")
  #list(APPEND CMAKE_PREFIX_PATH "$ENV{QT_SVGDIR}/Qt5Gui")
#endif()
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug)
  set(CMAKE_CXX_CPPCHECK "cppcheck")
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DDEBUG")
endif()
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -DNDEBUG")
if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  include_directories("/usr/local/include")
endif()
project(nvui)
if(MSVC)
  add_compile_options(/W4)
elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU")
  add_compile_options(-Wall -Wextra -pedantic)
endif()
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
find_package(msgpack CONFIG REQUIRED)
find_package(Catch2 CONFIG REQUIRED)
find_package(Qt5Core CONFIG REQUIRED)
find_package(Qt5Gui CONFIG REQUIRED)
find_package(Qt5Widgets CONFIG REQUIRED)
find_package(Qt5Svg CONFIG REQUIRED)
find_package(fmt CONFIG REQUIRED)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
file(GLOB SOURCES
  src/msgpack_overrides.hpp
  src/editor.hpp
  src/editor.cpp
  src/titlebar.hpp
  src/titlebar.cpp
  src/utils.cpp
  src/nvim.hpp
  src/nvim.cpp
  src/hlstate.hpp
  src/hlstate.cpp
  src/window.hpp
  src/window.cpp
  src/cursor.hpp
  src/cursor.cpp
  src/popupmenu.hpp
  src/popupmenu.cpp
  src/cmdline.hpp
  src/cmdline.cpp
  src/font.hpp
  src/grid.cpp
  src/grid.hpp
  src/decide_renderer.hpp
)
if (WIN32)
  message("Windows detected.")
  file(GLOB WINONLYSOURCES
    src/wineditor.hpp
    src/wineditor.cpp
    src/platform/windows/*.cpp
    src/platform/windows/*.hpp
  )
  list(APPEND SOURCES ${WINONLYSOURCES})
endif()
find_package(Boost COMPONENTS filesystem thread REQUIRED)
list(APPEND SOURCES "src/main.cpp")
if(WIN32 AND CMAKE_BUILD_TYPE STREQUAL "Release")
  add_executable(nvui WIN32 "assets/icons/desktop/neovim_icon.rc" ${SOURCES})
else()
  add_executable(nvui ${SOURCES})
endif()
target_link_libraries(nvui PRIVATE msgpackc msgpackc-cxx)
target_link_libraries(nvui PRIVATE Qt::Core Qt::Gui Qt::Widgets Qt::Svg)
target_link_libraries(nvui PRIVATE fmt::fmt-header-only)
target_include_directories(nvui PRIVATE
  "${PROJECT_SOURCE_DIR}/src"
)
if(WIN32)
  target_link_libraries(nvui PUBLIC
    d2d1.lib
    d3d11.lib
    dwrite.lib
  )
endif()
if (Boost_FOUND)
  target_include_directories(nvui PRIVATE ${Boost_INCLUDE_DIR})
endif()
target_link_libraries(${PROJECT_NAME} PRIVATE
  ${Boost_LIBRARIES}
)
include(CheckIPOSupported)
check_ipo_supported(RESULT LTOAvailable)
if(LTOAvailable)
  set_property(TARGET nvui PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()

file(GLOB TEST_SOURCES
  src/msgpack_overrides.hpp
  src/editor.hpp
  src/editor.cpp
  src/titlebar.hpp
  src/titlebar.cpp
  src/utils.hpp
  src/nvim.hpp
  src/nvim.cpp
  src/window.hpp
  src/window.cpp
  src/hlstate.hpp
  src/hlstate.cpp
  src/cursor.hpp
  src/cursor.cpp
  test/*.cpp
  src/popupmenu.hpp
  src/popupmenu.cpp
  src/cmdline.hpp
  src/cmdline.cpp
  src/font.hpp
  src/grid.cpp
  src/grid.hpp
  src/decide_renderer.hpp
)
if (WIN32)
  file(GLOB WINONLYTESTSOURCES
    src/wineditor.hpp
    src/wineditor.cpp
    src/platform/windows/*.cpp
    src/platform/windows/*.hpp
  )
  list(APPEND TEST_SOURCES ${WINONLYTESTSOURCES})
endif()
add_executable(nvui_test "test/test_main.cpp" ${TEST_SOURCES})
target_link_libraries(nvui_test PRIVATE msgpackc msgpackc-cxx)
target_link_libraries(nvui_test PRIVATE Catch2::Catch2)
target_link_libraries(nvui_test PRIVATE Qt::Core Qt::Gui Qt::Widgets Qt::Svg)
target_link_libraries(nvui_test PRIVATE fmt::fmt-header-only)
target_include_directories(nvui_test PRIVATE
  "${PROJECT_SOURCE_DIR}/test"
  "${PROJECT_SOURCE_DIR}/src"
)
if (BOOST_FOUND)
  target_include_directories(nvui_test PRIVATE ${Boost_INCLUDE_DIR})
endif()
target_link_libraries(nvui_test PRIVATE
  ${Boost_FILESYSTEM_LIBRARY}
  ${Boost_THREAD_LIBRARY}
  ${Boost_LIBRARIES}
)
include(CTest)
include(Catch)
catch_discover_tests(nvui_test)
