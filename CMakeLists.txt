cmake_minimum_required(VERSION 3.0)

project(theora VERSION 1.2.0)

option(BUILD_EXAMPLES "Build example programs" ON)
if(MSVC)
	option(MSVC_LINK_STATIC_RUNTIME "Use static linking of msvc runtime" OFF)
endif()

set(CMAKE_DEBUG_POSTFIX "d")

# switch from dynamic to static linking
if(MSVC AND MSVC_LINK_STATIC_RUNTIME)
	foreach(flag_var
		CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
		CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO)

		if(${flag_var} MATCHES "/MD")
			string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
		endif(${flag_var} MATCHES "/MD")
	endforeach(flag_var)
endif()

find_package(ogg REQUIRED CONFIG)

list(APPEND srcs
	include/theora/codec.h
	include/theora/theora.h
	include/theora/theoradec.h
	lib/internal.h
	lib/encint.h
	lib/enquant.h
	lib/huffenc.h
	lib/mathops.h
	lib/modedec.h
	lib/apiwrapper.h
	lib/bitpack.h
	lib/dct.h
	lib/decint.h
	lib/dequant.h
	lib/huffdec.h
	lib/huffman.h
	lib/idct.c
	lib/ocintrin.h
	lib/quant.h
	lib/analyze.c
	lib/encapiwrapper.c
	lib/encfrag.c
	lib/encinfo.c
	lib/encode.c
	lib/enquant.c
	lib/fdct.c
	lib/huffenc.c
	lib/mathops.c
	lib/mcenc.c
	lib/rate.c
	lib/tokenize.c
	lib/apiwrapper.c
	lib/bitpack.c
	lib/decapiwrapper.c
	lib/decinfo.c
	lib/decode.c
	lib/dequant.c
	lib/fragment.c
	lib/huffdec.c
	lib/idct.c
	lib/info.c
	lib/internal.c
	lib/quant.c
	lib/state.c
)

if(MSVC)
	list(APPEND compile_flags "/wd4554 /wd4700 /wd4244 /wd4018 /wd4267")
	list(APPEND srcs
		lib/x86_vc/x86cpu.h
		lib/x86_vc/mmxencfrag.c
		lib/x86_vc/mmxfdct.c
		lib/x86_vc/x86cpu.c
		lib/x86_vc/x86enc.c
		lib/x86_vc/mmxfrag.c
		lib/x86_vc/mmxidct.c
		lib/x86_vc/mmxstate.c
		lib/x86_vc/x86state.c)
	if ("${CMAKE_SIZEOF_VOID_P}" STREQUAL "4")
		list(APPEND compile_definitions
			OC_X86_ASM)
	endif()
endif()

# no shared library support in theora
add_library(libtheora STATIC ${srcs})
target_link_libraries(libtheora libogg)
set_target_properties(libtheora PROPERTIES
	COMPILE_FLAGS "${compile_flags}"
	COMPILE_DEFINITIONS "${compile_definitions}"
	INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/include"
	INTERFACE_INCLUDE_DIRECTORIES "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>;$<INSTALL_INTERFACE:include>"
	PREFIX ""
	IMPORT_PREFIX ""
	IMPORT_SUFFIX "_implib${CMAKE_IMPORT_LIBRARY_SUFFIX}"
	SOVERSION ${PROJECT_VERSION_MAJOR})

install(TARGETS libtheora
	EXPORT ${CMAKE_PROJECT_NAME}Config
	LIBRARY DESTINATION lib COMPONENT lib
	ARCHIVE DESTINATION lib COMPONENT lib
	RUNTIME DESTINATION bin COMPONENT bin)

install(DIRECTORY "include"
	DESTINATION . COMPONENT dev
	FILES_MATCHING
	PATTERN "*.h")

install(EXPORT ${CMAKE_PROJECT_NAME}Config
	DESTINATION cmake)

if(BUILD_EXAMPLES)
	list(APPEND encoder_example_srcs "examples/encoder_example.c")
	if(MSVC)
		list(APPEND examples_add_srcs "win32/getopt_win.h" "examples/getopt.h" "win32/getopt.c" "win32/getopt1.c")
		list(APPEND examples_compile_definitions "_CRT_SECURE_NO_WARNINGS")

	endif()

	add_executable(encoder_example ${encoder_example_srcs} ${examples_add_srcs})
	target_link_libraries(encoder_example libtheora)
	set_target_properties(encoder_example PROPERTIES
		COMPILE_FLAGS "${compile_flags}"
		COMPILE_DEFINITIONS "${examples_compile_definitions}"
		)

	if("${CMAKE_SYSTEM_NAME}" MATCHES "Linux")
		target_link_libraries(encoder_example m)
	endif()

	add_executable(dump_video examples/dump_video.c ${examples_add_srcs})
	target_link_libraries(dump_video libtheora)
	set_target_properties(dump_video PROPERTIES
		COMPILE_FLAGS "${compile_flags}"
		COMPILE_DEFINITIONS "${examples_compile_definitions}"
		)

	# just install these - don't export them
	install(TARGETS encoder_example dump_video
		RUNTIME DESTINATION bin COMPONENT bin)
endif()
