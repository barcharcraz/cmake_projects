CMAKE_MINIMUM_REQUIRED(VERSION 3.5)

#INCLUDE('cmake/HunterGate.cmake')
INCLUDE(ExternalProject)
PROJECT(deps)

SET_DIRECTORY_PROPERTIES(PROPERTIES EP_PREFIX "./base")
set(CMAKE_MSVCIDE_RUN_PATH <INSTALL_DIR>/bin)
SET(DEPS_BUILD_STATIC_MT ON CACHE BOOL "Build Static libs with MT libc")
SET(DEPS_BUILD_SHARED_MD ON CACHE BOOL "Build shared libs with MD libc")
SET(DEPS_BUILD_STATIC_MD OFF CACHE BOOL "Build static libs with MD libc")
SET(DEPS_BUILD_SHARED_MT OFF CACHE BOOL "Build shared libs with MT libc")
IF(${DEPS_BUILD_SHARED_MD} OR ${DEPS_BUILD_SHARED_MT})
	SET(DEPS_BUILD_SHARED ON)
ENDIF()
IF(${DEPS_BUILD_STATIC_MD} OR ${DEPS_BUILD_STATIC_MT})
	SET(DEPS_BUILD_STATIC ON)
ENDIF()
IF(${DEPS_BUILD_SHARED_MD} OR ${DEPS_BUILD_STATIC_MD})
	SET(DEPS_BUILD_MD ON)
ENDIF()
IF(${DEPS_BUILD_SHARED_MT} OR ${DEPS_BUILD_STATIC_MT})
	SET(DEPS_BUILD_MT ON)
ENDIF()
ExternalProject_Add(zlib
	URL http://zlib.net/zlib-1.2.8.tar.gz
	CMAKE_ARGS
		-DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
)
ExternalProject_Add(assimp
	URL https://github.com/assimp/assimp/archive/v3.2.tar.gz
	DEPENDS zlib
	CMAKE_ARGS
		-DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
		-DASSIMP_BUILD_ASSIMP_TOOLS=OFF
		-DASSIMP_BUILD_TESTS=OFF
		-DASSIMP_INSTALL_PDB=OFF
)

ExternalProject_Add(sqlite3
	GIT_REPOSITORY https://github.com/snikulov/sqlite.cmake.build.git
	CMAKE_ARGS
		-DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>

)
ExternalProject_Add(glfw
	URL https://github.com/glfw/glfw/releases/download/3.2/glfw-3.2.zip
	CMAKE_ARGS
		-DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
)
ExternalProject_Add(libtiff
	URL http://download.osgeo.org/libtiff/tiff-4.0.6.zip
	CMAKE_ARGS
		-DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
)
ExternalProject_Add(libpng
	URL http://download.sourceforge.net/libpng/lpng1622.zip
	CMAKE_ARGS
		-DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
)
ExternalProject_Add(OpenEXR
	URL http://download.savannah.nongnu.org/releases/openexr/openexr-2.2.0.tar.gz
	CMAKE_ARGS
		-DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
		-DILMBASE_PACKAGE_PREFIX=<INSTALL_DIR>
		-DCMAKE_MSVCIDE_RUN_PATH=<INSTALL_DIR>/lib
)
ExternalProject_Add(IlmBase
	URL http://download.savannah.nongnu.org/releases/openexr/ilmbase-2.2.0.tar.gz
	CMAKE_ARGS
		-DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
)
#SET(B2_ARGS STRING variant=$<JOIN:$<$<CONFIG:Debug>:debug> $<$<NOT:$<CONFIG:Debug>>:release>,$<COMMA>>
#	link=$<JOIN:$<$<TARGET_PROPERTY:DEPS_BUILD_STATIC>:static> $<$<TARGET_PROPERTY:DEPS_BUILD_SHARED>:shared>,$<COMMA>>
#	runtime-link
SET(B2_ARGS $<JOIN: $<$<CONFIG:Debug>:b2.exe stage --stagedir=<BINARY_DIR> variant=debug link=shared runtime-link=shared>
						   $<$<NOT:$<CONFIG:Debug>>:b2.exe stage --stagedir=<BINARY_DIR> variant=release link=shared runtime-link=shared>,
						    & >)
ExternalProject_Add(Boost
	URL https://sourceforge.net/projects/boost/files/boost/1.61.0/boost_1_61_0.7z
	URL_HASH SHA256=fa1c34862b7ba8674ed6e064a14667a11830c6252f702d9458451834b74f7815
	UPDATE_COMMAND ""
	DOWNLOAD_COMMAND ""
	#CONFIGURE_COMMAND cmd /C cd <SOURCE_DIR> & bootstrap.bat
	CONFIGURE_COMMAND cmd /C echo "${B2_ARGS}"
	BUILD_COMMAND cmd /C cd <SOURCE_DIR> & ${B2_ARGS}
	#BUILD_COMMAND cmd /C cd <SOURCE_DIR> & b2.exe stage --stagedir=<BINARY_DIR>
	INSTALL_COMMAND cmd /C cd <SOURCE_DIR> & b2.exe install --stagedir=<BINARY_DIR> --prefix=<INSTALL_DIR>
	LOG_CONFIGURE ON
	LOG_BUILD ON
)
ExternalProject_Add(OpenImageIO
	GIT_REPOSITORY https://github.com/OpenImageIO/oiio.git
	CMAKE_ARGS
		-DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
		-DBOOST_ROOT=<INSTALL_DIR>
)