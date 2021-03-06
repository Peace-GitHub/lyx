# This file is part of LyX, the document processor.
# Licence details can be found in the file COPYING.
#
# Copyright (c) 2012 Kornel Benko, kornel@lyx.org
#

project(supporttest)

macro(sources _program)
	set(_tmplist)
	foreach(_tmp ${ARGN})
		list(APPEND _tmplist "${TOP_SRC_DIR}/src/support/tests/${_tmp}")
	endforeach()
	set(${_program}_SOURCES ${_tmplist})
	add_executable(${_program} ${_tmplist})
	target_link_libraries(${_program} support ${intl_link}
		${Lyx_Boost_Libraries} ${QT_QTGUI_LIBRARY} ${QT_QTCORE_LIBRARY}
		${ZLIB_LIBRARY} ${ICONV_LIBRARY})
endmacro()

file(GLOB test_sources ${TOP_SRC_DIR}/src/support/tests/${LYX_CPP_FILES})

include_directories(
	${TOP_SRC_DIR}/src/support/tests
	${QT_INCLUDES}
	${ICONV_INCLUDE_DIR}
	${ZLIB_INCLUDE_DIR}
	${LIBINTL_INCLUDE_DIR})


set(check_PROGRAMS check_convert check_filetools check_lstrings)

file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/regfiles")

set(_depends)
foreach(_src ${check_PROGRAMS})
	sources(${_src} ${_src}.cpp dummy_functions.cpp boost.cpp)
	string(REPLACE "check_" "" _srcx ${_src})
	set(src_data_file "${TOP_SRC_DIR}/src/support/tests/regfiles/${_srcx}")
	set(created_data_file "${CMAKE_CURRENT_BINARY_DIR}/regfiles/${_src}_data")
        add_test(NAME build_${_srcx}
	  COMMAND "${CMAKE_COMMAND}" --build ${CMAKE_BINARY_DIR} --target ${_src})
	add_test(NAME data_${_srcx}
	  COMMAND ${_src} > "${created_data_file}")
	add_test(NAME compare_${_srcx}
	  COMMAND ${CMAKE_COMMAND} -E compare_files "${src_data_file}" "${created_data_file}")
	set_tests_properties(data_${_srcx} PROPERTIES DEPENDS build_${_srcx} )
	set_tests_properties(compare_${_srcx} PROPERTIES DEPENDS data_${_srcx})
endforeach()


