
add_definitions(-DHAVE_CONFIG_H=1)

# Without __NO_CTYPE tolower and friends are macros which introduce a GLIBC 2.3
# dependency. By defining this identifier we are currently backwards compatible
# to GLIBC 2.1.3, which is a good thing. In addition, the macros lead to code
# which triggers warnings with -Wunreachable-code.
if(HAVE_GLIBC)
	add_definitions(-D__NO_CTYPE=1)
endif()


if(HAVE_GCC_VISIBILITY)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fvisibility=hidden")
	add_definitions(-DHAVE_GCC_VISIBILITY)
endif()

# So our static library can be linked into shared libraries (for instance,
# gcc on amd64 needs -fPIC)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${CMAKE_SHARED_LIBRARY_C_FLAGS}")

set(ALUT_SOURCES
	alutBufferData.c
	alutCodec.c
	alutError.c
	alutInit.c
	alutInputStream.c
	alutLoader.c
	alutOutputStream.c
	alutUtil.c
	alutVersion.c
	alutWaveform.c)

set(ALUT_INTERNAL_HEADERS
	alutInternal.h)
source_group(InternalHeader FILES ${ALUT_INTERNAL_HEADERS})

set(ALUT_HEADERS
	../include/AL/alut.h)
source_group(APIHeader FILES ${ALUT_HEADERS})

add_library(alut ${ALUT_SOURCES} ${ALUT_INTERNAL_HEADERS} ${ALUT_HEADERS})
set_property(TARGET
	alut
	PROPERTY
	PUBLIC_HEADER
	${ALUT_HEADERS})
set_target_properties(alut
	PROPERTIES
	VERSION
	${MAJOR_VERSION}
	SOVERSION
	${MAJOR_VERSION})
target_link_libraries(alut ${OPENAL_LIBRARY})
target_include_directories(alut PUBLIC ${OPENAL_INCLUDE_DIR})
if(UNIX)
	target_link_libraries(alut m)
endif()	
if(NOT BUILD_SHARED_LIBS)
    target_compile_definitions(alut PUBLIC ALUT_BUILD_STATIC)
endif()

install(TARGETS alut
	EXPORT "FreeALUTTargets"
	LIBRARY DESTINATION lib${LIB_SUFFIX}
	ARCHIVE DESTINATION lib${LIB_SUFFIX}
	RUNTIME DESTINATION bin
	PUBLIC_HEADER DESTINATION include/AL
	)

include(CMakePackageConfigHelpers)

set(version_config "${CMAKE_CURRENT_BINARY_DIR}/temp/FreeALUTConfigVersion.cmake")
set(project_config "${CMAKE_CURRENT_BINARY_DIR}/temp/FreeALUTConfig.cmake")
set(namespace "FreeALUT::")

write_basic_package_version_file("${version_config}"
									COMPATIBILITY SameMajorVersion
									VERSION ${PACKAGE_VERSION})

configure_package_config_file("Config.cmake.in"
								"${project_config}"
								INSTALL_DESTINATION
								lib${LIB_SUFFIX}/cmake/freealut)

install(FILES "${project_config}" "${version_config}"
		DESTINATION lib${LIB_SUFFIX}/cmake/freealut)

install(EXPORT "FreeALUTTargets"
		NAMESPACE "${namespace}"
		DESTINATION lib${LIB_SUFFIX}/cmake/freealut)
