# raptor/CMakeLists.txt
#
# This file allows building Raptor2 with the CMake configuration tool.
# Download CMake in source or binary form from http://www.cmake.org/
#
# Original listfile by Daniel Richard G. <skunk@iSKUNK.ORG>
# This file is in the public domain.
#

SET(RAPTOR_VERSION_MAJOR	2)
SET(RAPTOR_VERSION_MINOR	0)
SET(RAPTOR_VERSION_RELEASE	16)
SET(RAPTOR_MIN_VERSION_DECIMAL	20016)

SET(VERSION "${RAPTOR_VERSION_MAJOR}.${RAPTOR_VERSION_MINOR}.${RAPTOR_VERSION_RELEASE}")
MATH(EXPR RAPTOR_VERSION_DECIMAL "${RAPTOR_VERSION_MAJOR} * 10000 + ${RAPTOR_VERSION_MINOR} * 100 + ${RAPTOR_VERSION_RELEASE}")

PROJECT(Raptor2 C)

CMAKE_MINIMUM_REQUIRED(VERSION 2.8.7)

INCLUDE( GNUInstallDirs )

SET(CMAKE_INCLUDE_CURRENT_DIR 1)

################################################################

# Third-party libraries/dependencies

FIND_PACKAGE(CURL)
FIND_PACKAGE(LibXml2)
FIND_PACKAGE(LibXslt)
#FIND_PACKAGE(YAJL)
FIND_PACKAGE(Perl  REQUIRED)
FIND_PACKAGE(BISON 3 REQUIRED)
FIND_PACKAGE(FLEX  REQUIRED)

if(EXISTS ${CURL_INCLUDE_DIRS})
  INCLUDE_DIRECTORIES(${CURL_INCLUDE_DIRS})
endif(EXISTS ${CURL_INCLUDE_DIRS})

if(EXISTS ${EXPAT_INCLUDE_DIRS})
  INCLUDE_DIRECTORIES(${EXPAT_INCLUDE_DIRS})
endif(EXISTS ${EXPAT_INCLUDE_DIRS})

if(EXISTS ${LIBXML2_INCLUDE_DIR})
  INCLUDE_DIRECTORIES(${LIBXML2_INCLUDE_DIR})
endif(EXISTS ${LIBXML2_INCLUDE_DIR})

if(EXISTS ${LIBXSLT_INCLUDE_DIRS})
  INCLUDE_DIRECTORIES(${LIBXSLT_INCLUDE_DIRS})
endif(EXISTS ${LIBXSLT_INCLUDE_DIRS})

################################################################

# Configuration checks

INCLUDE(CheckIncludeFile)
INCLUDE(CheckIncludeFiles)
INCLUDE(CheckFunctionExists)
INCLUDE(CheckTypeSize)
INCLUDE(CheckStructHasMember)
INCLUDE(CheckCSourceCompiles)

CHECK_INCLUDE_FILE(errno.h	HAVE_ERRNO_H)
CHECK_INCLUDE_FILE(fcntl.h	HAVE_FCNTL_H)
CHECK_INCLUDE_FILE(getopt.h	HAVE_GETOPT_H)
CHECK_INCLUDE_FILE(limits.h	HAVE_LIMITS_H)
CHECK_INCLUDE_FILE(math.h	HAVE_MATH_H)
CHECK_INCLUDE_FILE(setjmp.h	HAVE_SETJMP_H)
CHECK_INCLUDE_FILE(stddef.h	HAVE_STDDEF_H)
CHECK_INCLUDE_FILE(stdlib.h	HAVE_STDLIB_H)
CHECK_INCLUDE_FILE(string.h	HAVE_STRING_H)
CHECK_INCLUDE_FILE(unistd.h	HAVE_UNISTD_H)
CHECK_INCLUDE_FILE(sys/param.h	HAVE_SYS_PARAM_H)
CHECK_INCLUDE_FILE(sys/stat.h	HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILE(sys/stat.h	HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILE(sys/time.h	HAVE_SYS_TIME_H)

CHECK_INCLUDE_FILES("sys/time.h;time.h" TIME_WITH_SYS_TIME)

CHECK_FUNCTION_EXISTS(access		HAVE_ACCESS)
CHECK_FUNCTION_EXISTS(_access		HAVE__ACCESS)
CHECK_FUNCTION_EXISTS(getopt		HAVE_GETOPT)
CHECK_FUNCTION_EXISTS(getopt_long	HAVE_GETOPT_LONG)
CHECK_FUNCTION_EXISTS(gettimeofday	HAVE_GETTIMEOFDAY)
CHECK_FUNCTION_EXISTS(isascii		HAVE_ISASCII)
CHECK_FUNCTION_EXISTS(setjmp		HAVE_SETJMP)
CHECK_FUNCTION_EXISTS(snprintf		HAVE_SNPRINTF)
CHECK_FUNCTION_EXISTS(_snprintf		HAVE__SNPRINTF)
CHECK_FUNCTION_EXISTS(stat		HAVE_STAT)
CHECK_FUNCTION_EXISTS(strcasecmp	HAVE_STRCASECMP)
CHECK_FUNCTION_EXISTS(stricmp		HAVE_STRICMP)
CHECK_FUNCTION_EXISTS(_stricmp		HAVE__STRICMP)
CHECK_FUNCTION_EXISTS(strtok_r		HAVE_STRTOK_R)
CHECK_FUNCTION_EXISTS(vasprintf		HAVE_VASPRINTF)
CHECK_FUNCTION_EXISTS(vsnprintf		HAVE_VSNPRINTF)
CHECK_FUNCTION_EXISTS(_vsnprintf	HAVE__VSNPRINTF)

CHECK_TYPE_SIZE("unsigned char"		SIZEOF_UNSIGNED_CHAR)
CHECK_TYPE_SIZE("unsigned short"	SIZEOF_UNSIGNED_SHORT)
CHECK_TYPE_SIZE("unsigned int"		SIZEOF_UNSIGNED_INT)
CHECK_TYPE_SIZE("unsigned long"		SIZEOF_UNSIGNED_LONG)
CHECK_TYPE_SIZE("unsigned long long"	SIZEOF_UNSIGNED_LONG_LONG)

CHECK_C_SOURCE_COMPILES("
int main(void){ printf(__FUNCTION__); }"
	HAVE___FUNCTION__
)


IF(LIBXML2_FOUND)

	SET(CMAKE_REQUIRED_INCLUDES  ${LIBXML2_INCLUDE_DIR})
	SET(CMAKE_REQUIRED_LIBRARIES ${LIBXML2_LIBRARIES})

	CHECK_FUNCTION_EXISTS(xmlCtxtUseOptions     HAVE_XMLCTXTUSEOPTIONS)
	CHECK_FUNCTION_EXISTS(xmlSAX2InternalSubset HAVE_XMLSAX2INTERNALSUBSET)

	CHECK_STRUCT_HAS_MEMBER(
		xmlEntity
		name_length
		libxml/parser.h
		RAPTOR_LIBXML_ENTITY_NAME_LENGTH
	)

	CHECK_STRUCT_HAS_MEMBER(
		xmlEntity
		etype
		libxml/parser.h
		RAPTOR_LIBXML_ENTITY_ETYPE
	)

	CHECK_STRUCT_HAS_MEMBER(
		xmlSAXHandler
		initialized
		libxml/parser.h
		RAPTOR_LIBXML_XMLSAXHANDLER_INITIALIZED
	)

	CHECK_STRUCT_HAS_MEMBER(
		xmlSAXHandler
		externalSubset
		libxml/parser.h
		RAPTOR_LIBXML_XMLSAXHANDLER_EXTERNALSUBSET
	)

	CHECK_C_SOURCE_COMPILES("
#include <libxml/parser.h>
int main(void){ xmlParserOption foo; foo = XML_PARSE_NONET; return 0; }"
		RAPTOR_LIBXML_XML_PARSE_NONET
	)

	CHECK_C_SOURCE_COMPILES("
#include <libxml/HTMLparser.h>
int main(void){ htmlParserOption foo; foo = HTML_PARSE_NONET; return 0; }"
		RAPTOR_LIBXML_HTML_PARSE_NONET
	)

	SET(CMAKE_REQUIRED_INCLUDES)
	SET(CMAKE_REQUIRED_LIBRARIES)

ENDIF(LIBXML2_FOUND)

################################################################

# User-configurable options
#
# (Note: The CMake GUI displays these in alphabetical order, regardless of
# the order we use here)

SET(BUILD_SHARED_LIBS OFF CACHE BOOL
	"Build shared libraries instead of static ones.")

IF(BUILD_SHARED_LIBS)
	SET(RAPTOR_STATIC FALSE)
ELSE(BUILD_SHARED_LIBS)
	SET(RAPTOR_STATIC TRUE)
ENDIF(BUILD_SHARED_LIBS)

SET(RAPTOR_XML_INIT none)
IF(LIBXML2_FOUND)
	SET(RAPTOR_XML_INIT libxml)
ENDIF(LIBXML2_FOUND)

SET(RAPTOR_XML ${RAPTOR_XML_INIT} CACHE STRING
	"Which XML library to use (any of \"libxml\", \"none\").")

SET(RAPTOR_XML_DEFINE RAPTOR_XML_NONE)
IF(RAPTOR_XML STREQUAL "libxml")
	SET(RAPTOR_XML_DEFINE RAPTOR_XML_LIBXML)
ENDIF(RAPTOR_XML STREQUAL "libxml")

SET(RAPTOR_WWW_INIT none)
IF(CURL_FOUND)
	SET(RAPTOR_WWW_INIT curl)
ENDIF(CURL_FOUND)
IF(LIBXML2_FOUND)
	SET(RAPTOR_WWW_INIT xml)
ENDIF(LIBXML2_FOUND)

SET(RAPTOR_WWW ${RAPTOR_WWW_INIT} CACHE STRING
	"Which WWW library to use (any of \"curl\", \"xml\", \"none\").")

SET(RAPTOR_WWW_DEFINE RAPTOR_WWW_NONE)
IF(RAPTOR_WWW STREQUAL "curl")
	SET(RAPTOR_WWW_DEFINE RAPTOR_WWW_LIBCURL)
ELSEIF(RAPTOR_WWW STREQUAL "xml")
	SET(RAPTOR_WWW_DEFINE RAPTOR_WWW_LIBXML)
ENDIF(RAPTOR_WWW STREQUAL "curl")

SET(RAPTOR_XML_1_1 FALSE CACHE BOOL
	"Use XML version 1.1 name checking.")

SET(HAVE_RAPTOR_PARSE_DATE 1)
SET(RAPTOR_PARSEDATE 1)

SET(RAPTOR_PARSER_RDFXML_INIT FALSE)
IF(LIBXML2_FOUND)
	SET(RAPTOR_PARSER_RDFXML_INIT TRUE)
ENDIF(LIBXML2_FOUND)

SET(RAPTOR_PARSER_GRDDL_INIT FALSE)
IF(LIBXML2_FOUND AND LIBXSLT_FOUND)
	SET(RAPTOR_PARSER_GRDDL_INIT TRUE)
ENDIF(LIBXML2_FOUND AND LIBXSLT_FOUND)

SET(RAPTOR_PARSER_RDFXML ${RAPTOR_PARSER_RDFXML_INIT}
	CACHE BOOL "Build RDF/XML parser.")
SET(RAPTOR_PARSER_NTRIPLES TRUE
	CACHE BOOL "Build N-Triples parser.")
SET(RAPTOR_PARSER_TURTLE TRUE
	CACHE BOOL "Build Turtle parser.")
SET(RAPTOR_PARSER_TRIG TRUE
	CACHE BOOL "Build TRiG parser.")
SET(RAPTOR_PARSER_RSS ${LIBXML2_FOUND}
	CACHE BOOL "Build RSS Tag Soup parser.")
SET(RAPTOR_PARSER_GRDDL ${RAPTOR_PARSER_GRDDL_INIT}
	CACHE BOOL "Build GRDDL parser.")
SET(RAPTOR_PARSER_GUESS TRUE
	CACHE BOOL "Build guess parser.")
SET(RAPTOR_PARSER_RDFA ${LIBXML2_FOUND}
	CACHE BOOL "Build RDFA parser.")
SET(RAPTOR_PARSER_JSON ${YAJL_FOUND}
	CACHE BOOL "Build JSON parser.")
SET(RAPTOR_PARSER_NQUADS TRUE
	CACHE BOOL "Build N-Quads parser.")

SET(RAPTOR_SERIALIZER_RDFXML TRUE
	CACHE BOOL "Build RDF/XML serializer.")
SET(RAPTOR_SERIALIZER_NTRIPLES TRUE
	CACHE BOOL "Build N-Triples serializer.")
SET(RAPTOR_SERIALIZER_RDFXML_ABBREV TRUE
	CACHE BOOL "Build RDF/XML-abbreviated serializer.")
SET(RAPTOR_SERIALIZER_TURTLE TRUE
	CACHE BOOL "Build Turtle serializer.")
SET(RAPTOR_SERIALIZER_RSS_1_0 TRUE
	CACHE BOOL "Build RSS 1.0 serializer.")
SET(RAPTOR_SERIALIZER_ATOM TRUE
	CACHE BOOL "Build Atom 1.0 serializer.")
SET(RAPTOR_SERIALIZER_DOT TRUE
	CACHE BOOL "Build GraphViz DOT serializer.")
SET(RAPTOR_SERIALIZER_HTML TRUE
	CACHE BOOL "Build HTML Table serializer.")
SET(RAPTOR_SERIALIZER_JSON TRUE
	CACHE BOOL "Build JSON serializer.")
SET(RAPTOR_SERIALIZER_NQUADS TRUE
	CACHE BOOL "Build N-Quads serializer.")

################################################################

CONFIGURE_FILE(
	src/raptor_config_cmake.h.in
	src/raptor_config.h
	@ONLY
)

ADD_DEFINITIONS(-DHAVE_CONFIG_H)

CONFIGURE_FILE(
	src/raptor2.h.in
	src/raptor2.h
	@ONLY
)

################################################################

message(STATUS "
################################################################
Raptor Configuration Summary
################################################################

    http://librdf.org/raptor/

    Configured on host $ENV{COMPUTERNAME} ${HOSTNAME}
      host OS                   ${CMAKE_SYSTEM_NAME}
      host architecture         ${CMAKE_SYSTEM_PROCESSOR}

    General flags:
      CC                        ${CMAKE_C_COMPILER}
      CXX (not used)            ${CMAKE_CXX_COMPILER}
      CFLAGS                    ${CMAKE_C_FLAGS}
      CXXFLAGS                  ${CMAKE_CXX_FLAGS}
      LDFLAGS                   ${CMAKE_EXE_LINKER_FLAGS}

    Installation prefix:        ${CMAKE_INSTALL_PREFIX}

    Dependencies (packages marked with *** are required):
    "
)

message( "  *** Perl                        ${PERL_EXECUTABLE}")
message( "  *** Bison                       ${BISON_EXECUTABLE}")
message( "  *** Flex                        ${FLEX_EXECUTABLE}")
message( "  *** libxml2 (lib)               ${LIBXML2_LIBRARIES}")
message( "  *** libxml2 (include)           ${LIBXML2_INCLUDE_DIR}")

################################################################

# On Windows, the test/utility programs usually have DLL dependencies that
# make them cumbersome to invoke within IDE environments, so we enable the
# test suite only when makefiles are being used.
#
# Note that a partial solution is described at
#
#	http://www.cmake.org/pipermail/cmake/2009-May/029464.html
#
# but we don't have a good way of knowing where the third-party DLLs are.
#
IF(CMAKE_GENERATOR MATCHES "Makefiles")
	ENABLE_TESTING()
ENDIF(CMAKE_GENERATOR MATCHES "Makefiles")

SUBDIRS(src)
SUBDIRS(utils)

################################################################

FIND_PROGRAM(JING jing)
IF(NOT JING STREQUAL "JING-NOTFOUND")
	SET(HAVE_JING TRUE)
ENDIF(NOT JING STREQUAL "JING-NOTFOUND")

SET(RAPPER  ${CMAKE_BINARY_DIR}/utils/rapper)
SET(RDFDIFF ${CMAKE_BINARY_DIR}/utils/rdfdiff)

MACRO(RAPPER_TEST NAME_PREFIX COMMAND OUTPUT REFERENCE)
	FILE(WRITE ${CMAKE_CURRENT_BINARY_DIR}/test-${NAME_PREFIX}.cmake "
EXECUTE_PROCESS(
	COMMAND ${COMMAND}
	TIMEOUT 10
	OUTPUT_FILE ${OUTPUT}.raw
	ERROR_QUIET
)

# http://public.kitware.com/Bug/view.php?id=13007

CONFIGURE_FILE(
	${CMAKE_CURRENT_BINARY_DIR}/${OUTPUT}.raw
	${OUTPUT}
	@ONLY
	NEWLINE_STYLE UNIX
)
")
	ADD_TEST("${NAME_PREFIX}:run" ${CMAKE_COMMAND} -P test-${NAME_PREFIX}.cmake)
	ADD_TEST("${NAME_PREFIX}:cmp" ${CMAKE_COMMAND} -E compare_files ${REFERENCE} ${OUTPUT})
ENDMACRO(RAPPER_TEST NAME_PREFIX COMMAND OUTPUT REFERENCE)

INCLUDE(CMakeParseArguments)

FUNCTION(RAPPER_RDFDIFF_TEST TEST_NAME RAPPER_COMMAND RAPPER_OUTPUT RDFDIFF_COMMAND)
  # http://www.cmake.org/pipermail/cmake/2012-July/051309.html
  SET(options EXPECTED_FAILURE)
  SET(oneValueArgs )
  SET(multiValueArgs )
  CMAKE_PARSE_ARGUMENTS(RRT "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN})

  SET(TEST_NAME ${ARGV0})
  SET(RAPPER_COMMAND ${ARGV1})
  SET(RAPPER_OUTPUT ${ARGV2})
  SET(RDFDIFF_COMMAND ${ARGV3})

  # workaround because TRUE is not true in a CMake IF
  IF(RRT_EXPECTED_FAILURE)
    SET(EXPECT_FAILURE 1)
  ELSE()
    SET(EXPECT_FAILURE 0)
  ENDIF()

	FILE(WRITE ${CMAKE_CURRENT_BINARY_DIR}/test-${TEST_NAME}.cmake "
SET(rdfdiff_status -1)

EXECUTE_PROCESS(
	COMMAND ${RAPPER_COMMAND}
	TIMEOUT 10
	OUTPUT_FILE ${RAPPER_OUTPUT}
	ERROR_QUIET
	RESULT_VARIABLE rapper_status
)

IF(rapper_status EQUAL 0 OR rapper_status EQUAL 2)
	EXECUTE_PROCESS(
		COMMAND ${RDFDIFF_COMMAND}
		TIMEOUT 10
		OUTPUT_QUIET
		ERROR_QUIET
		RESULT_VARIABLE rdfdiff_status
	)
ENDIF(rapper_status EQUAL 0 OR rapper_status EQUAL 2)

IF(${EXPECT_FAILURE})
  IF(rdfdiff_status EQUAL 0)
    MESSAGE(FATAL_ERROR \"Test succeeded, but expected to fail.\")
  ENDIF(rdfdiff_status EQUAL 0)
ELSE()
  IF(NOT rdfdiff_status EQUAL 0)
    MESSAGE(FATAL_ERROR \"Test failed.\")
  ENDIF(NOT rdfdiff_status EQUAL 0)
ENDIF()
")

	ADD_TEST(${TEST_NAME} ${CMAKE_COMMAND} -P test-${TEST_NAME}.cmake)
ENDFUNCTION(RAPPER_RDFDIFF_TEST)

SUBDIRS(tests/feeds)
SUBDIRS(tests/grddl)
SUBDIRS(tests/json)
SUBDIRS(tests/ntriples)
SUBDIRS(tests/rdfa)
SUBDIRS(tests/rdfa11)
SUBDIRS(tests/rdfxml)
SUBDIRS(tests/trig)
#SUBDIRS(tests/turtle)	# TODO

# end raptor/CMakeLists.txt
