cmake_minimum_required(VERSION 3.12)
include(${CMAKE_CURRENT_SOURCE_DIR}/HunterGate.cmake)
#set(HUNTER_RUN_INSTALL ON)
HunterGate(
    URL "https://github.com/cpp-pm/hunter/archive/v0.23.312.tar.gz"
    SHA1 "42311324f6b859983d833cffc2151d0b86b8bea4"
    FILEPATH "${CMAKE_CURRENT_SOURCE_DIR}/../Snap/WarpEngine/cmake/HunterConfig.cmake"
)

project(ARToolkit)

hunter_add_package(Eigen)
find_package(Eigen3 REQUIRED)

hunter_add_package(OpenCV)
find_package(OpenCV REQUIRED)
include_directories(${OpenCV_DIR}/include)

hunter_add_package(OpenSceneGraph)
find_package(OpenSceneGraph CONFIG REQUIRED)
include_directories(${OPENSCENEGRAPH_INCLUDE_DIRS})

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include ${CMAKE_CURRENT_SOURCE_DIR}/lib/SRC/KPM/FreakMatcher/)

get_target_property(EIGEN_INCLUDE_DIRECTORY Eigen3::Eigen INTERFACE_INCLUDE_DIRECTORIES)
include_directories(${EIGEN_INCLUDE_DIRECTORY} ${EIGEN_INCLUDE_DIRECTORY}/unsupported)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/include/AR/config.h.in ${CMAKE_CURRENT_SOURCE_DIR}/include/AR/config.h)
list(APPEND ARTOOLKIT_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/include/AR/config.h")

include(${CMAKE_CURRENT_SOURCE_DIR}/filelist.cmake)

set(ARTOOLKIT_SOURCES_EXTRA "")
set(ARTOOLKIT_VIDEO_SOURCES_EXTRA "")
if(APPLE)
  string(COMPARE EQUAL "${CMAKE_OSX_SYSROOT}" "iphoneos" _is_ios)
  string(COMPARE EQUAL "${CMAKE_OSX_SYSROOT}" "iphonesimulator" _is_ios_sim)
  if(_is_ios OR _is_ios_sim)
    set(ARTOOLKIT_SOURCES_EXTRA ${ARTOOLKIT_GL_SOURCES_MOBILE})
    set(ARTOOLKIT_VIDEO_SOURCES_EXTRA ${ARTOOLKIT_VIDEO_SOURCES_IPHONE})
  endif()
  if(CMAKE_OSX_SYSROOT MATCHES ".*MacOSX.*")
    set(_is_osx TRUE)
    set(ARTOOLKIT_SOURCES_EXTRA ${ARTOOLKIT_GL_SOURCES_DESKTOP})
    set(ARTOOLKIT_VIDEO_SOURCES_EXTRA ${ARTOOLKIT_VIDEO_SOURCES_MACOSX})
  endif()
elseif(ANDROID)
  set(_is_android TRUE)
  set(ARTOOLKIT_SOURCES_EXTRA ${ARTOOLKIT_GL_SOURCES_MOBILE})
  set(ARTOOLKIT_VIDEO_SOURCES_EXTRA ${ARTOOLKIT_VIDEO_SOURCES_ANDROID})
  add_definitions(-DANDROID -DHAVE_PTHREADS)
  #Android video needs cURL to get the calibration from server
  hunter_add_package(CURL)
  find_package(CURL CONFIG REQUIRED)
  include_directories(${CMAKE_CURRENT_LIST_DIR}/android/jni/androidcamera/include)
elseif(UNIX) #assume Linux
  set(_is_linux TRUE)
  set(ARTOOLKIT_SOURCES_EXTRA ${ARTOOLKIT_GL_SOURCES_DESKTOP})
  set(ARTOOLKIT_VIDEO_SOURCES_EXTRA ${ARTOOLKIT_VIDEO_SOURCES_LINUX})
elseif(WIN32)
  set(_is_windows TRUE)
  set(ARTOOLKIT_SOURCES_EXTRA ${ARTOOLKIT_GL_SOURCES_DESKTOP})
  set(ARTOOLKIT_VIDEO_SOURCES_EXTRA ${ARTOOLKIT_VIDEO_SOURCES_WINDOWS})
endif()

add_library(ARToolkit STATIC ${ARTOOLKIT_SOURCES} ${ARTOOLKIT_KPM_SOURCES} ${ARTOOLKIT_SOURCES_EXTRA})
add_library(ARToolkitVideo STATIC ${ARTOOLKIT_VIDEO_SOURCES} ${ARTOOLKIT_VIDEO_SOURCES_EXTRA})

if(ANDROID)
  target_link_libraries(ARToolkitVideo CURL::libcurl)
endif()

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)

install(TARGETS ARToolkit ARToolkitVideo
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)
