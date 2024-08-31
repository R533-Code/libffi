cmake_minimum_required(VERSION 3.5)
project(libffi VERSION 3.4.6 LANGUAGES C ASM)

set(SOURCES_LIST
    #src/closures.c
    #src/java_raw_api.c
    src/prep_cif.c
    src/raw_api.c
    src/types.c
    $<$<OR:$<CONFIG:Debug>,$<CONFIG:RelWithDebInfo>>:src/debug.c>
)
if(BUILD_TYPE STREQUAL "Debug" OR BUILD_TYPE STREQUAL "RelWithDebInfo")
    add_compile_definitions(FFI_DEBUG)
endif()

include(configure_platform.cmake)
include(configure_options.cmake)

file(COPY ${FFI_CONFIG_FILE} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY src/${TARGETDIR}/ffitarget.h DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/include)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/include)
include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(include)

add_library(ffi ${SOURCES_LIST})
target_compile_definitions(ffi PRIVATE FFI_BUILDING)
set_property(TARGET ffi PROPERTY POSITION_INDEPENDENT_CODE 1)

target_include_directories(ffi PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/include)
target_include_directories(ffi PUBLIC ${CMAKE_CURRENT_BINARY_DIR})
target_include_directories(ffi PUBLIC include)

if(MSVC)
    if (${BUILD_SHARED_LIBS})
      target_compile_definitions(ffi PRIVATE FFI_BUILDING_DLL)
    endif()
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
elseif(CMAKE_SYSTEM_NAME MATCHES Darwin)
    set(CMAKE_MACOSX_RPATH 1)
endif()

set_target_properties(ffi PROPERTIES OUTPUT_NAME libffi)
add_library(ffi::ffi ALIAS ffi)