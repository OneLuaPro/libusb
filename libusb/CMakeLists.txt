# ------------------------------------------------------------------------------
# lfs DLL
add_library(libusb STATIC)
# setup platform-specific sources, compile and linker options
if(WIN32 AND NOT MinGW)
  target_compile_options(libusb PRIVATE /D_WIN32 /D_CRT_SECURE_NO_WARNINGS)
  # target_link_options(lfs PRIVATE /LIBPATH:${LIBLUA_LIBDIR} liblua.lib)
else()
  message(FATAL_ERROR "Not yet fully implemented.")
endif()
# plattform-independend sources
target_sources(libusb
  PRIVATE
  os/events_windows.h os/events_windows.c os/threads_windows.h os/threads_windows.c
  os/windows_common.h os/windows_common.c os/windows_usbdk.h os/windows_usbdk.c
  os/windows_winusb.h os/windows_winusb.c
  libusbi.h version.h version_nano.h 
  core.c descriptor.c hotplug.c io.c strerror.c sync.c
  libusbi.h
  libusb-1.0.rc
  PUBLIC FILE_SET HEADERS
  FILES libusb.h
)
# special includes for config.h
target_include_directories(libusb
  PRIVATE
  ${CMAKE_SOURCE_DIR}/msvc
  ${CMAKE_SOURCE_DIR}/libusb
)

# ------------------------------------------------------------------------------
# Installation instructions
install(
  TARGETS libusb FILE_SET HEADERS
  RUNTIME DESTINATION ${INSTALL_BINDIR} COMPONENT bin
  LIBRARY DESTINATION ${INSTALL_LIBDIR} COMPONENT lib
  ARCHIVE DESTINATION ${INSTALL_LIBDIR} COMPONENT lib
  PUBLIC_HEADER DESTINATION ${INSTALL_INCLUDEDIR} COMPONENT dev
)
