cmake_minimum_required(VERSION 3.4)

if(WIN32)
  # Check for required Environment variables
  if(NOT DEFINED ENV{VCPKG_ROOT})
    # set(VCPKG_ROOT "C:\\repo\\vcpkg_latest")
    message(FATAL_ERROR "VCPKG_ROOT is not defined. Set the location of vcpkg")
  endif()

  set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake")
else()  # UNIX
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/CMakeModules")
endif()

project(GameplayFootball)
set(PACKAGE GameplayFootball)

set (CMAKE_CXX_STANDARD 14)

if(UNIX)
   set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g -Wall -O3")
   # Temporarily suppress warnings during debug
   set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wno-unused-variable -Wno-sign-compare -Wno-unused-function -Wno-strict-aliasing")
   set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -Wall -O3")

   SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${GCC_COVERAGE_COMPILE_FLAGS} -fPIC -g -O3")
   SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC -O3 -g")
endif(UNIX)

if(MSVC AND DEFINED ENV{MAX_OPTIMIZATION})
   set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2 -GL")
   set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -LTCG")
   set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -LTCG")
   set(CMAKE_STATIC_LINKER_FLAGS "${CMAKE_STATIC_LINKER_FLAGS} -LTCG")
endif()

# Find required libraries
FIND_PACKAGE(OpenGL REQUIRED)
include_directories(${OPENGL_INCLUDE_DIR})

FIND_PACKAGE(SDL2 REQUIRED)
include_directories(${SDL2_INCLUDE_DIR})

if(WIN32)
   FIND_PACKAGE(SDL2-image REQUIRED)
   FIND_PACKAGE(SDL2-ttf REQUIRED)
   FIND_PACKAGE(SDL2-gfx REQUIRED)
else()  # Unix
   FIND_PACKAGE(SDL2_image REQUIRED)
   FIND_PACKAGE(SDL2_ttf REQUIRED)
   FIND_PACKAGE(SDL2_gfx REQUIRED)
endif()

include_directories(${SDL2_IMAGE_DIRS})
include_directories(${SDL2_TTF_DIRS})
include_directories(${SDL2_GFX_DIRS})

FIND_PACKAGE(Boost REQUIRED COMPONENTS thread system filesystem)
include_directories(${Boost_INCLUDE_DIR})

FIND_PACKAGE(OpenAL REQUIRED)
include_directories(${OPENAL_INCLUDE_DIR})

FIND_PACKAGE(SQLite3 REQUIRED)
include_directories(${SQLite3_INCLUDE_DIRS})

include_directories(${PROJECT_SOURCE_DIR}/src)

# Include the sources
include(sources.cmake)

# Compile it as multiple static libraries (note: not compiling physics,
# as not used by gameplayfootball)

add_library(baselib OBJECT ${BASE_SOURCES} ${BASE_HEADERS}
        ${BASE_GEOMETRY_HEADERS} ${BASE_MATH_HEADERS})
add_library(systemscommonlib OBJECT ${SYSTEMS_COMMON_SOURCES}
        ${SYSTEMS_COMMON_HEADERS})
add_library(systemsgraphicslib OBJECT ${SYSTEMS_GRAPHICS_SOURCES}
        ${SYSTEMS_GRAPHICS_HEADERS} ${SYSTEMS_GRAPHICS_OBJECTS_HEADERS}
        ${SYSTEMS_GRAPHICS_RESOURCES_HEADERS} ${SYSTEMS_GRAPHICS_RENDERING_HEADERS})
add_library(systemsaudiolib OBJECT ${SYSTEMS_AUDIO_SOURCES}
        ${SYSTEMS_AUDIO_HEADERS} ${SYSTEMS_AUDIO_OBJECTS_HEADERS}
        ${SYSTEMS_AUDIO_RESOURCES_HEADERS} ${SYSTEMS_AUDIO_RENDERING_HEADERS})
add_library(loaderslib OBJECT ${LOADERS_SOURCES} ${LOADERS_HEADERS})
add_library(typeslib OBJECT ${TYPES_SOURCES} ${TYPES_HEADERS})
add_library(frameworklib OBJECT ${FRAMEWORK_SOURCES} ${FRAMEWORK_HEADERS})
add_library(scenelib OBJECT ${SCENE_SOURCES} ${SCENE_HEADERS}
        ${SCENE2D_HEADERS} ${SCENE3D_HEADERS} ${SCENE_OBJECTS_HEADERS}
        ${SCENE_RESOURCES_HEADERS})
add_library(managerslib OBJECT ${MANAGERS_SOURCES} ${MANAGERS_HEADERS})
add_library(utilslib OBJECT ${UTILS_SOURCES} ${UTILS_HEADERS}
        ${UTILS_EXT_HEADERS})
add_library(gui2lib OBJECT ${UTILS_GUI2_SOURCES} ${UTILS_GUI2_HEADERS}
        ${UTILS_GUI2_WIDGETS_HEADERS})

# Join all created static libraries into a single static or shared one.
set(OWN_LIBRARIES $<TARGET_OBJECTS:baselib> $<TARGET_OBJECTS:systemscommonlib>
        $<TARGET_OBJECTS:systemsgraphicslib>
        $<TARGET_OBJECTS:systemsaudiolib> $<TARGET_OBJECTS:loaderslib>
        $<TARGET_OBJECTS:typeslib> $<TARGET_OBJECTS:frameworklib>
        $<TARGET_OBJECTS:scenelib> $<TARGET_OBJECTS:managerslib>
        $<TARGET_OBJECTS:utilslib> $<TARGET_OBJECTS:gui2lib>)

# Compile it as multiple static libraries
add_library(blunted2 ${BLUNTED_CORE_SOURCES} ${BLUNTED_CORE_HEADERS} ${OWN_LIBRARIES})
add_library(leaguelib ${LEAGUE_SOURCES} ${LEAGUE_HEADERS})
add_library(gamelib ${GAME_SOURCES} ${GAME_HEADERS})
add_library(hidlib ${HID_SOURCES} ${HID_HEADERS})
add_library(menulib ${MENU_SOURCES} ${MENU_HEADERS})
add_library(datalib ${DATA_SOURCES} ${DATA_HEADERS})

if(UNIX)
    set(SDL2_LIBS_ALL ${SDL2_IMAGE_LIBRARIES} ${SDL2_TTF_LIBRARIES} ${SDL2_GFX_LIBRARIES} ${SDL2_LIBRARIES})
elseif(WIN32)
    set(SDL2_LIBS_ALL SDL2::SDL2 SDL2::SDL2_image SDL2::SDL2_ttf SDL2::SDL2_gfx SDL2::SDL2main)
endif()

set(LIBRARIES gamelib hidlib menulib datalib leaguelib blunted2
    Boost::filesystem Boost::system Boost::thread SQLite::SQLite3
    ${SDL2_LIBS_ALL} ${OPENAL_LIBRARY} ${OPENGL_LIBRARY})


add_executable(gameplayfootball WIN32 ${CORE_SOURCES} ${CORE_HEADERS})
target_link_libraries(gameplayfootball ${LIBRARIES})
