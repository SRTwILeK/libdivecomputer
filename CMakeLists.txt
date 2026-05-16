cmake_minimum_required(VERSION 3.10)
project(libdivecomputer C RC)

# ── version.h (generated from version.h.in at configure time) ────────────────
set(DC_VERSION       "0.10.0-devel")
set(DC_VERSION_MAJOR 0)
set(DC_VERSION_MINOR 10)
set(DC_VERSION_MICRO 0)
configure_file(
    include/libdivecomputer/version.h.in
    ${CMAKE_CURRENT_BINARY_DIR}/include/libdivecomputer/version.h
    @ONLY
)

# ── revision.h (git HEAD hash, captured at configure time) ───────────────────
find_package(Git QUIET)
if(GIT_FOUND)
    execute_process(
        COMMAND ${GIT_EXECUTABLE} rev-parse HEAD
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        OUTPUT_VARIABLE GIT_REVISION
        OUTPUT_STRIP_TRAILING_WHITESPACE
        ERROR_QUIET
    )
endif()
if(NOT GIT_REVISION)
    set(GIT_REVISION "unknown")
endif()
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/revision.h
    "#define DC_VERSION_REVISION \"${GIT_REVISION}\"\n"
)

# ── libusb (static, built from source — no CMakeLists.txt upstream) ───────────
set(LIBUSB_DIR "${CMAKE_CURRENT_SOURCE_DIR}/deps/libusb")

file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/libusb-config")
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/libusb-config/config.h"
"/* config.h for libusb — MinGW/GCC Windows build */\n"
"#define DEFAULT_VISIBILITY /**/\n"
"#define PRINTF_FORMAT(a, b) __attribute__((format(printf, a, b)))\n"
"#define ENABLE_LOGGING 1\n"
"#define PLATFORM_WINDOWS 1\n"
)

add_library(libusb-1.0 STATIC
    ${LIBUSB_DIR}/libusb/core.c
    ${LIBUSB_DIR}/libusb/descriptor.c
    ${LIBUSB_DIR}/libusb/hotplug.c
    ${LIBUSB_DIR}/libusb/io.c
    ${LIBUSB_DIR}/libusb/strerror.c
    ${LIBUSB_DIR}/libusb/sync.c
    ${LIBUSB_DIR}/libusb/os/events_windows.c
    ${LIBUSB_DIR}/libusb/os/threads_windows.c
    ${LIBUSB_DIR}/libusb/os/windows_common.c
    ${LIBUSB_DIR}/libusb/os/windows_usbdk.c
    ${LIBUSB_DIR}/libusb/os/windows_winusb.c
)

target_include_directories(libusb-1.0
    PUBLIC  "${LIBUSB_DIR}/libusb"
    PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/libusb-config"
)

target_compile_definitions(libusb-1.0 PRIVATE _WIN32_WINNT=0x0600)

# ── hidapi (static, add_subdirectory — Windows backend uses native HID API) ───
set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
set(HIDAPI_WITH_TESTS OFF CACHE BOOL "" FORCE)
set(HIDAPI_INSTALL_TARGETS OFF CACHE BOOL "" FORCE)
add_subdirectory(deps/hidapi EXCLUDE_FROM_ALL)

# ── export .def file generated from symbols list ──────────────────────────────
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/src/libdivecomputer.symbols" _raw_symbols)
set(_def_file "${CMAKE_CURRENT_BINARY_DIR}/libdivecomputer.def")
file(WRITE  ${_def_file} "EXPORTS\n")
foreach(_sym ${_raw_symbols})
    string(STRIP "${_sym}" _sym)
    if(_sym)
        file(APPEND ${_def_file} "${_sym}\n")
    endif()
endforeach()

# ── libdivecomputer (shared DLL) ──────────────────────────────────────────────
add_library(libdivecomputer SHARED
    src/aes.c
    src/array.c
    src/atomics_cobalt.c
    src/atomics_cobalt_parser.c
    src/ble.c
    src/bluetooth.c
    src/buffer.c
    src/checksum.c
    src/citizen_aqualand.c
    src/citizen_aqualand_parser.c
    src/cochran_commander.c
    src/cochran_commander_parser.c
    src/common.c
    src/context.c
    src/cressi_edy.c
    src/cressi_edy_parser.c
    src/cressi_goa.c
    src/cressi_goa_parser.c
    src/cressi_leonardo.c
    src/cressi_leonardo_parser.c
    src/custom.c
    src/datetime.c
    src/deepblu_cosmiq.c
    src/deepblu_cosmiq_parser.c
    src/deepsix_excursion.c
    src/deepsix_excursion_parser.c
    src/descriptor.c
    src/device.c
    src/diverite_nitekq.c
    src/diverite_nitekq_parser.c
    src/divesoft_freedom.c
    src/divesoft_freedom_parser.c
    src/divesystem_idive.c
    src/divesystem_idive_parser.c
    src/halcyon_symbios.c
    src/halcyon_symbios_parser.c
    src/hdlc.c
    src/hw_frog.c
    src/hw_ostc.c
    src/hw_ostc3.c
    src/hw_ostc_parser.c
    src/ihex.c
    src/iostream.c
    src/irda.c
    src/iterator.c
    src/liquivision_lynx.c
    src/liquivision_lynx_parser.c
    src/mares_common.c
    src/mares_darwin.c
    src/mares_darwin_parser.c
    src/mares_iconhd.c
    src/mares_iconhd_parser.c
    src/mares_nemo.c
    src/mares_nemo_parser.c
    src/mares_puck.c
    src/mclean_extreme.c
    src/mclean_extreme_parser.c
    src/oceanic_atom2.c
    src/oceanic_atom2_parser.c
    src/oceanic_common.c
    src/oceanic_veo250.c
    src/oceanic_veo250_parser.c
    src/oceanic_vtpro.c
    src/oceanic_vtpro_parser.c
    src/oceans_s1.c
    src/oceans_s1_common.c
    src/oceans_s1_parser.c
    src/packet.c
    src/parser.c
    src/pelagic_i330r.c
    src/platform.c
    src/rbstream.c
    src/reefnet_sensus.c
    src/reefnet_sensuspro.c
    src/reefnet_sensuspro_parser.c
    src/reefnet_sensusultra.c
    src/reefnet_sensusultra_parser.c
    src/reefnet_sensus_parser.c
    src/ringbuffer.c
    src/seac_screen.c
    src/seac_screen_common.c
    src/seac_screen_parser.c
    src/serial_win32.c
    src/shearwater_common.c
    src/shearwater_petrel.c
    src/shearwater_predator.c
    src/shearwater_predator_parser.c
    src/socket.c
    src/sporasub_sp2.c
    src/sporasub_sp2_parser.c
    src/suunto_common.c
    src/suunto_common2.c
    src/suunto_d9.c
    src/suunto_d9_parser.c
    src/suunto_eon.c
    src/suunto_eonsteel.c
    src/suunto_eonsteel_parser.c
    src/suunto_eon_parser.c
    src/suunto_solution.c
    src/suunto_solution_parser.c
    src/suunto_vyper.c
    src/suunto_vyper2.c
    src/suunto_vyper_parser.c
    src/tecdiving_divecomputereu.c
    src/tecdiving_divecomputereu_parser.c
    src/timer.c
    src/usb.c
    src/usbhid.c
    src/uwatec_aladin.c
    src/uwatec_memomouse.c
    src/uwatec_memomouse_parser.c
    src/uwatec_smart.c
    src/uwatec_smart_parser.c
    src/version.c
    src/zeagle_n2ition3.c
    src/libdivecomputer.rc
)

target_include_directories(libdivecomputer PRIVATE
    ${CMAKE_CURRENT_BINARY_DIR}/include  # generated version.h
    include                               # public headers
    src
    ${CMAKE_CURRENT_BINARY_DIR}          # generated revision.h
)

target_compile_definitions(libdivecomputer PRIVATE
    WIN32
    NDEBUG
    _WINDOWS
    _USRDLL
    LIBDIVECOMPUTER_EXPORTS
    ENABLE_LOGGING
    HAVE_VERSION_SUFFIX
    HAVE_AF_IRDA_H
    HAVE_WS2BTH_H
    HAVE_LIBUSB
    HAVE_HIDAPI
    __USE_MINGW_ANSI_STDIO=1
)

target_link_options(libdivecomputer PRIVATE
    "${_def_file}"
    -static-libgcc
)

target_link_libraries(libdivecomputer PRIVATE
    ws2_32
    libusb-1.0
    hidapi_winapi
)

set_target_properties(libdivecomputer PROPERTIES
    OUTPUT_NAME "libdivecomputer"
    PREFIX ""
)
