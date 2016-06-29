set(LIBNU_SRC
	cesu8.c
	ducet.c
	extra.c
	strcoll.c
	strings.c
	tofold.c
	tolower.c
	toupper.c
	utf16.c
	utf16be.c
	utf16he.c
	utf16le.c
	utf32.c
	utf32be.c
	utf32he.c
	utf32le.c
	utf8.c
	validate.c
	version.c
)

set(INCLUDES
    casemap.h
    casemap_internal.h
    cesu8.h
    cesu8_internal.h
    config.h
    defines.h
    ducet.h
    extra.h
    libnu.h
    mph.h
    strcoll.h
    strcoll_internal.h
    strings.h
    udb.h
    utf16be.h
    utf16.h
    utf16he.h
    utf16_internal.h
    utf16le.h
    utf32be.h
    utf32.h
    utf32he.h
    utf32_internal.h
    utf32le.h
    utf8.h
    utf8_internal.h
    validate.h
    version.h
)

if(BUILD_SHARED_LIBS)
    add_library(${LIB_NAME} SHARED ${LIBNU_SRC})
    set_target_properties(${LIB_NAME} PROPERTIES SOVERSION 3)
#    set_target_properties(${LIB_NAME} PROPERTIES DEFINE_SYMBOL JBIG_DLL_EXPORTS)
else()
    add_definitions(-DJBIG_STATIC)
    add_library(${LIB_NAME} STATIC ${LIBNU_SRC})
    set_target_properties(${LIB_NAME} PROPERTIES SOVERSION 3)
    #set_target_properties(${LIB_NAME} PROPERTIES DEFINE_SYMBOL JBIG_STATIC)
endif()

target_link_extlibraries(${LIB_NAME})

target_include_directories(${LIB_NAME} PUBLIC
	$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
	$<INSTALL_INTERFACE:include>)

target_compile_definitions(${LIB_NAME} PUBLIC
	${NU_BUILD_OPTIONS}
	$<$<OR:$<CONFIG:Release>,$<CONFIG:PROF>>:NU_DISABLE_CONTRACTIONS>
)

add_custom_target(uninstall COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

if(NOT CYGWIN)
    set_target_properties(${LIB_NAME} PROPERTIES VERSION ${VERSION})
endif()

set_target_properties(${LIB_NAME} PROPERTIES DEBUG_POSTFIX d)
if(UNIX)
    # On unix-like platforms the library is almost always called lib${PROJECT_NAME}
#    set_target_properties(${LIB_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
    set_target_properties(${LIB_NAME} PROPERTIES OUTPUT_NAME nu)
endif()
if(NOT UNIX)
	set_target_properties(nu PROPERTIES OUTPUT_NAME libnu)
endif()

install(FILES ${INCLUDES} DESTINATION "${INSTALL_INC_DIR}/libnu")
if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL )
    install(TARGETS ${LIB_NAME}
        DESTINATION "${INSTALL_LIB_DIR}")
endif()
