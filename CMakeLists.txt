cmake_minimum_required(VERSION 2.8)
cmake_policy(SET CMP0015 NEW)

# Project Definition
project(odhcpd C)

set(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g3 -std=gnu99")

FIND_PATH(ubox_include_dir libubox/uloop.h)
FIND_PATH(libnl-tiny_include_dir netlink-generic.h PATH_SUFFIXES libnl-tiny)
INCLUDE_DIRECTORIES(${ubox_include_dir} ${libnl-tiny_include_dir})

FIND_LIBRARY(libnl NAMES nl-tiny)

add_definitions(-D_GNU_SOURCE -Os -Wall -Werror --std=gnu99)
IF(CMAKE_C_COMPILER_VERSION VERSION_GREATER 6)
	add_definitions(-Wextra -Werror=implicit-function-declaration)
	add_definitions(-Wformat -Werror=format-security -Werror=format-nonliteral)
ENDIF()
add_definitions(-Wno-unused-parameter -Wmissing-declarations)


if (${EXT_CER_ID})
	add_definitions(-DEXT_CER_ID=${EXT_CER_ID})
endif(${EXT_CER_ID})

if(${UBUS})
	add_definitions(-DWITH_UBUS)
	set(EXT_SRC ${EXT_SRC} src/ubus.c)
	set(EXT_LINK ${EXT_LINK} ubus)
endif(${UBUS})

if(${DHCPV4_SUPPORT})
	add_definitions(-DDHCPV4_SUPPORT)
	set(EXT_SRC ${EXT_SRC} src/dhcpv4.c)
endif(${DHCPV4_SUPPORT})

add_executable(odhcpd src/odhcpd.c src/config.c src/router.c src/dhcpv6.c src/ndp.c src/dhcpv6-ia.c src/dhcpv6-pxe.c src/netlink.c ${EXT_SRC})
target_link_libraries(odhcpd resolv ubox uci ${libnl} ${EXT_LINK})

# Installation
install(TARGETS odhcpd DESTINATION sbin/)


# Packaging information
set(CPACK_PACKAGE_VERSION "1")
set(CPACK_PACKAGE_CONTACT "Steven Barth <steven@midlink.org>")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "odhcpd")
set(CPACK_GENERATOR "DEB;RPM;STGZ")
set(CPACK_STRIP_FILES true)

SET(CPACK_DEBIAN_PACKAGE_VERSION ${CPACK_PACKAGE_VERSION})
set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}_${CPACK_DEBIAN_PACKAGE_VERSION}")

include(CPack)

