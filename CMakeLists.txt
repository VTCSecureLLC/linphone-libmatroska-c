############################################################################
# CMakeLists.txt
# Copyright (C) 2016  Belledonne Communications, Grenoble France
#
############################################################################
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
#
############################################################################

cmake_minimum_required(VERSION 3.0)

project(matroska2
	VERSION 0.22.1
	LANGUAGES C
)

option(ENABLE_SHARED "Build shared library." YES)
option(ENABLE_STATIC "Build static library." YES)
option(CONFIG_EBML_WRITING "Enable EBML file writing support" ON)
option(CONFIG_EBML_UNICODE "Enable Unicode support" ON)
option(CONFIG_DEBUGCHECKS "Enable internal checks" ON)
option(CONFIG_STDIO "Use stdio.h for writing" ON)
option(CONFIG_FILEPOS_64 "Encode filepos_t on 64bits" ON)
option(CONFIG_DEBUG_LEAKS "Enable internal menory leaks checking support" OFF)

include(GNUInstallDirs)

if(NOT CMAKE_INSTALL_RPATH AND CMAKE_INSTALL_PREFIX)
	set(CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_FULL_LIBDIR})
	message(STATUS "Setting install rpath to ${CMAKE_INSTALL_RPATH}")
endif()

configure_file(config.h.cmake ${CMAKE_BINARY_DIR}/config.h)

include_directories(
	${CMAKE_BINARY_DIR}
	corec
)

set(LINK_FLAGS )
if(APPLE)
	list(APPEND LINK_FLAGS "-framework Foundation")
	if(IOS)
		list(APPEND LINK_FLAGS "-framework CoreFoundation")
	endif()
endif()
string(REPLACE ";" " " LINK_FLAGS_STR "${LINK_FLAGS}")

if(WIN32)
	add_definitions(
		"-DEBML2_EXPORTS"
		"-DMATROSKA2_EXPORTS"
		"-DSTR_EXPORTS"
	)
endif()
add_compile_options(-w)

add_subdirectory(corec/corec)
add_subdirectory(libebml2)
add_subdirectory(libmatroska2)

install(FILES ${CMAKE_BINARY_DIR}/config.h
	DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/corec
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
	${CMAKE_BINARY_DIR}/Matroska2ConfigVersion.cmake
	VERSION ${MATROSKA2_VERSION}
	COMPATIBILITY AnyNewerVersion
)

export(EXPORT Matroska2Targets
	FILE "${CMAKE_CURRENT_BINARY_DIR}/Matroska2Targets.cmake"
)

configure_file(Matroska2Config.cmake.in
	${CMAKE_BINARY_DIR}/Matroska2Config.cmake
	@ONLY
)

set(CONFIG_PACKAGE_LOCATION "${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/cmake")
install(EXPORT Matroska2Targets
	FILE Matroska2Targets.cmake
	DESTINATION ${CONFIG_PACKAGE_LOCATION}
)
install(FILES
	${CMAKE_BINARY_DIR}/Matroska2ConfigVersion.cmake
	${CMAKE_BINARY_DIR}/Matroska2Config.cmake
	DESTINATION ${CONFIG_PACKAGE_LOCATION}
)
