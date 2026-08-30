# MIT License for CMake Build Infrastructure. See COPYING for the license of
# libusb itself.
#
# Copyright (c) 2023-2026 The OneLuaPro project authors.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy of
# this software and associated documentation files (the "Software"), to deal in
# the Software without restriction, including without limitation the rights to
# use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
# the Software, and to permit persons to whom the Software is furnished to do so,
# subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
#

# ------------------------------------------------------------------------------
# Generate version_describe.h
# Define the output directory and file path inside the build tree
get_filename_component(LIBUSB_GEN_INCLUDE_DIR
  "${CMAKE_CURRENT_BINARY_DIR}/gen_include" ABSOLUTE)
set(VERSION_DESCRIBE_H "${LIBUSB_GEN_INCLUDE_DIR}/version_describe.h")

# Determine the correct host script execution command
if(CMAKE_HOST_WIN32)
  find_program(POWERSHELL_EXE NAMES powershell pwsh REQUIRED)
  set(GEN_DESCRIBE_COMMAND "${POWERSHELL_EXE}" -NoProfile -ExecutionPolicy Bypass 
    -File "${CMAKE_SOURCE_DIR}/build-aux/gen-describe.ps1"
  )
else()
  set(GEN_DESCRIBE_COMMAND "sh" "${CMAKE_SOURCE_DIR}/build-aux/gen-describe.sh")
endif()

# Create a custom rule to generate the header file
add_custom_command(
  OUTPUT "${VERSION_DESCRIBE_H}"
  COMMAND ${CMAKE_COMMAND} -E make_directory "${LIBUSB_GEN_INCLUDE_DIR}"
  COMMAND ${GEN_DESCRIBE_COMMAND} "${CMAKE_SOURCE_DIR}" "${VERSION_DESCRIBE_H}"
  WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
  COMMENT "Generating version_describe.h via libusb script..."
  VERBATIM
)

# ------------------------------------------------------------------------------
# libusb DLL
add_library(libusb STATIC)
# setup platform-specific sources, compile and linker options
if(WIN32 AND NOT MinGW)
  target_compile_definitions(libusb PRIVATE
    _WIN32 _CRT_SECURE_NO_WARNINGS LIBUSB_WINDOWS_HOTPLUG)
  if(CMAKE_C_COMPILER_ID MATCHES "IntelLLVM")
    target_compile_options(libusb PRIVATE
      -Wno-unused-value
      -Wno-pragma-pack
      -Wno-unused-parameter
    )
  endif()
else()
  message(FATAL_ERROR "Not yet fully implemented.")
endif()
# plattform-independend sources
target_sources(libusb PRIVATE
  os/events_windows.h os/events_windows.c os/threads_windows.h os/threads_windows.c
  os/windows_common.h os/windows_common.c os/windows_usbdk.h os/windows_usbdk.c
  os/windows_winusb.h os/windows_winusb.c
  os/windows_hotplug.h os/windows_hotplug.c
  libusbi.h version.h version_nano.h 
  core.c descriptor.c hotplug.c io.c strerror.c sync.c
  libusbi.h
  libusb-1.0.rc
  # Include the generated header tracking file as a source dependency
  "${VERSION_DESCRIBE_H}"
  PUBLIC FILE_SET HEADERS
  FILES libusb.h
)
# special includes for config.h
target_include_directories(libusb
  PRIVATE
  ${CMAKE_SOURCE_DIR}/msvc
  ${CMAKE_SOURCE_DIR}/libusb
  # Add the generated include path so the compiler finds version_describe.h
  "${LIBUSB_GEN_INCLUDE_DIR}"
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
