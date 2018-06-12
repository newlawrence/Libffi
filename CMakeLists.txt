cmake_minimum_required(VERSION 3.5)
project(libffi C)
enable_language(ASM_MASM)
set(VERSION 3.2.1)

set(FFI_CLOSURES 1)
set(FFI_DEBUG 0)
set(FFI_NO_STRUCTS 0)
set(FFI_NO_RAW_API 0)
set(FFI_EXEC_TRAMPOLINE_TABLE 0)

set(TARGET X86)
set(HAVE_LONG_DOUBLE 0)
set(HAVE_LONG_DOUBLE_VARIANT 0)

message(STATUS "Setting build type to '${CMAKE_BUILD_TYPE}'")
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(HAVE_64BIT 1)
    set(HAVE_AS_X86_64_UNWIND_SECTION_TYPE 1)
    message(STATUS "Build configured for x86-64 architecture")
else()
    set(HAVE_64BIT 0)
    message(STATUS "Build configured for x86 architecture")
endif()

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/lib")
set(inc "${CMAKE_SOURCE_DIR}/include")
set(src "${CMAKE_SOURCE_DIR}/source")
set(bin "${CMAKE_SOURCE_DIR}/build")

if (MSVC)
    set(SYMBOL_UNDERSCORE 1)
    set(HAVE_HIDDEN_VISIBILITY_ATTRIBUTE 0)
    set(sources
        "${src}/closures.c"
        "${src}/prep_cif.c"
        "${src}/raw_api.c"
        "${src}/types.c"
    )

    if(HAVE_64BIT)
        set(TARGET X86_WIN64)
        set(HAVE_AS_X86_64_UNWIND_SECTION_TYPE 1)
        set(sources "${sources}" "${src}/ffi64.c")

        configure_file("${src}/asm/win64.S" "${bin}/source/win64.S.i" COPYONLY)
        set(pre "${bin}/source/win64.S.i")
        set(asm "${bin}/source/win64.S")
        set(fmasm "/c /Cx /Fo")
    else()
        set(TARGET X86_WIN32)
        set(HAVE_AS_X86_64_UNWIND_SECTION_TYPE 0)
        set(sources "${sources}" "${src}/ffi.c")

        configure_file("${src}/asm/win32.S" "${bin}/source/win32.S.i" COPYONLY)
        set(pre "${bin}/source/win32.S.i")
        set(asm "${bin}/source/win32.S")
        set(fmasm "/c /Cx /coff /safeseh /Fo")
    endif()

    set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /Oy- /Oi /Gy /Zi /GL /Gd")
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /Oy-")

    add_custom_target(preprocess
        COMMAND ${CMAKE_C_COMPILER} ${pre} /I ${bin}/include /EP > ${asm}
        SOURCES ${pre}
    )
endif()

include_directories("${bin}/include")
configure_file("${inc}/ffi.h.in" "${bin}/include/ffi.h")
configure_file("${inc}/fficonfig.h.in" "${bin}/include/fficonfig.h")
configure_file("${inc}/ffitarget.h" "${bin}/include/ffitarget.h" COPYONLY)
configure_file("${inc}/ffi_common.h" "${bin}/include/ffi_common.h" COPYONLY)

set_source_files_properties(${sources} PROPERTIES LANGUAGE C)
set_source_files_properties(${asm}
PROPERTIES
    LANGUAGE ASM_MASM
    GENERATED TRUE
    COMPILE_FLAGS ${fmasm}
)
add_library(ffi ${sources} ${asm})
set(CMAKE_STATIC_LINKER_FLAGS "${CMAKE_STATIC_LINKER_FLAGS} /LTCG")
target_compile_definitions(ffi PUBLIC "FFI_BUILDING")
set_target_properties(ffi PROPERTIES PREFIX "lib")
add_dependencies(ffi preprocess)
