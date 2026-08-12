## This file is a quick tutorial on writing CMakeLists for targeting the Vita
cmake_minimum_required(VERSION 3.16)

## This includes the Vita toolchain, must go before project definition
# It is a convenience so you do not have to type
# -DCMAKE_TOOLCHAIN_FILE=$VITASDK/share/vita.toolchain.cmake for cmake. It is
# highly recommended that you include this block for all projects.
if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "toolchain file")
  else()
    message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
  endif()
endif()

## Define project parameters here
# Name of the project
project(th06-vita)
# This line adds Vita helper macros, must go after project definition in order
# to build Vita specific artifacts (self/vpk).
include("${VITASDK}/share/vita.cmake" REQUIRED)

## Configuration options for this app
# Display name (under bubble in LiveArea)
set(VITA_APP_NAME "Touhou Koumakyou")
# Unique ID must be exactly 9 characters. Recommended: XXXXYYYYY where X =
# unique string of developer and Y = a unique number for this app
set(VITA_TITLEID  "TOHO06REV")
# Optional version string to show in LiveArea's more info screen
set(VITA_VERSION  "01.00")

find_package(SDL2 REQUIRED)

## Flags and includes for building
# Note that we make sure not to overwrite previous flags
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++20")
# set(CMAKE_CXX_FLAGS_RELEASE "-O3")
# Optional. You can specify more param.sfo flags this way.
set(VITA_MKSFOEX_FLAGS "${VITA_MKSFOEX_FLAGS} -d PARENTAL_LEVEL=1")

include_directories(
  src
)

include_directories(
    $ENV{VITASDK}/arm-vita-eabi/include
    $ENV{VITASDK}/arm-vita-eabi/include/SDL2
)

add_compile_definitions(PLAT_VITA)

add_executable(${PROJECT_NAME}
    "src/AnmManager.cpp"
    "src/AsciiManager.cpp"
    "src/BombData.cpp"
    "src/BulletData.cpp"
    "src/BulletManager.cpp"
    "src/Chain.cpp"
    "src/Controller.cpp"
    "src/EclManager.cpp"
    "src/EffectManager.cpp"
    "src/Ending.cpp"
    "src/EnemyEclInstr.cpp"
    "src/EnemyManager.cpp"
    "src/FileSystem.cpp"
    "src/GameErrorContext.cpp"
    "src/GameManager.cpp"
    "src/GameWindow.cpp"
    "src/Gui.cpp"
    "src/ItemManager.cpp"
    "src/main.cpp"
    "src/MainMenu.cpp"
    "src/MidiOutput.cpp"
    "src/MusicRoom.cpp"
    "src/Player.cpp"
    "src/ReplayManager.cpp"
    "src/ResultScreen.cpp"
    "src/Rng.cpp"
    "src/ScreenEffect.cpp"
    "src/SoundPlayer.cpp"
    "src/Stage.cpp"
    "src/Supervisor.cpp"
    "src/TextHelper.cpp"
    "src/ZunMath.cpp"
    "src/utils.cpp"
    "src/ZunTimer.cpp"
    "src/graphics/GLFunc.cpp"
    "src/graphics/FixedFunctionGL.cpp"
    "src/graphics/WebGL.cpp"
    "src/graphics/Software.cpp"
    "src/pbg3/FileAbstraction.cpp"
    "src/pbg3/IPbg3Parser.cpp"
    "src/pbg3/Pbg3Archive.cpp"
    "src/pbg3/Pbg3Parser.cpp"
    "src/thirdparty/sjis_converter.cpp"
    "src/midi/MidiDefault.cpp"
)

# Library to link to (drop the -l prefix). This will mostly be stubs.
target_link_libraries(${PROJECT_NAME}
-Wl,--whole-archive pthread -Wl,--no-whole-archive
          SDL2
          SDL2_image
          SDL2_ttf
          freetype
          zip
          png
          webp
          webpdemux
          bz2
          vitaGL
          vitashark
          SceShaccCgExt
          SceShaccCg_stub
          stdc++
          mathneon
          OpenSLES
          m
          c
          z
          taihen_stub
          SceAppMgr_stub
          SceCtrl_stub
          SceKernelDmacMgr_stub
          SceGxm_stub
          SceCommonDialog_stub
          SceLibKernel_stub
          SceAudio_stub
          SceTouch_stub
          SceHid_stub
          SceMotion_stub
          SceSysmodule_stub
          SceIofilemgr_stub
          SceNetCtl_stub
          SceNet_stub
          SceDisplay_stub
          SceAppUtil_stub
          SceAudioIn_stub
          SceIofilemgr_stub
          ScePower_stub
          SceProcessmgr_stub
          SceIme_stub
)

## Create Vita files
vita_create_self(${PROJECT_NAME}.self ${PROJECT_NAME})
# The FILE directive lets you add additional files to the VPK, the syntax is
# FILE src_path dst_path_in_vpk. In this case, we add the LiveArea paths.
vita_create_vpk(${PROJECT_NAME}.vpk ${VITA_TITLEID} ${PROJECT_NAME}.self
  VERSION ${VITA_VERSION}
  NAME ${VITA_APP_NAME}
  FILE resources/vita/sce_sys/icon0.png sce_sys/icon0.png
  FILE resources/vita/sce_sys/livearea/contents/bg.png sce_sys/livearea/contents/bg.png
  FILE resources/vita/sce_sys/livearea/contents/startup.png sce_sys/livearea/contents/startup.png
  FILE resources/vita/sce_sys/livearea/contents/template.xml sce_sys/livearea/contents/template.xml
)

