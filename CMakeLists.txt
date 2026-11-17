# The Empire Strikes Back (ESB.CVP): Atari's 1985 arcade game on the
# ColecoVision. The Star Wars plugin's sources built with SW_ESB: the same
# board with the game's ROM banked three ways, one of them by the
# 137412-101 slapstic, from the user's own ROMs on the card (/ESB). RESET
# goes back to the menu.
#
#   cmake -B build -G Ninja -DPICO_SDK_PATH=$HOME/pico-sdk
#   ninja -C build                      -> build/ESB.CVP
#
# Copy ESB.CVP into the card's /PLUGINS folder.
cmake_minimum_required(VERSION 3.13)

# the cart's SDK: the copy in sdk/ (sdk/README.md)
set(PARROTCART_SDK ${CMAKE_CURRENT_LIST_DIR}/sdk/firmware/revc)
set(PICO_BOARD_HEADER_DIRS ${PARROTCART_SDK}/cvcart_revc/boards)
set(PICO_BOARD cvcart_revc)
set(PICO_PLATFORM rp2350-arm-s)
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()
# the USB host: sdk/tinyusb (TinyUSB 0.21 and its fixes since), not the pico-sdk's 0.18
include(${PARROTCART_SDK}/sdk/cvpad_tinyusb.cmake)
include(${CMAKE_CURRENT_LIST_DIR}/cmake/pico_sdk_import.cmake)

project(esb C CXX ASM)
pico_sdk_init()
include(${PARROTCART_SDK}/sdk/cvplugin.cmake)

# The same source makes the same file, whenever it's built: the compiler's
# __DATE__ and __TIME__ (the build date in the binary info) come from
# SOURCE_DATE_EPOCH, fixed here at the tested build's, 2026-11-15 19:28:57 UTC.
set(BUILD_EPOCH 1794770937)
set(CMAKE_C_COMPILER_LAUNCHER ${CMAKE_COMMAND} -E env SOURCE_DATE_EPOCH=${BUILD_EPOCH})
set(CMAKE_CXX_COMPILER_LAUNCHER ${CMAKE_COMMAND} -E env SOURCE_DATE_EPOCH=${BUILD_EPOCH})

set(OURS src/starwars.c src/swcard.c src/swzip.c src/sw.c src/swcv.c src/tms5220.c)
set_source_files_properties(${OURS} PROPERTIES COMPILE_OPTIONS "-Wall;-Wextra")

add_executable(esb ${OURS} ${PARROTCART_SDK}/midihello/vdp.c)
target_include_directories(esb PRIVATE ${CMAKE_CURRENT_LIST_DIR}/src ${PARROTCART_SDK}/midihello)
target_compile_definitions(esb PRIVATE SW_ESB PICO_PRINTF_SUPPORT_FLOAT=0 PICO_PRINTF_SUPPORT_EXPONENTIAL=0)
cvframes_use(esb)
cvsd_use(esb)
cvpad_use(esb)                              # USB game pads: the port is a host
pico_enable_stdio_usb(esb 0)
pico_enable_stdio_uart(esb 0)
cvcart_plugin(esb ID parrotcart.esb NAME "Empire Strikes Back" VERSION 0.1.3 BY "Parrot Cart"
              HOME /ESB USES usb-host ICON src/esb_icon.txt FILE ESB.CVP README CART-README.md)
