# Minimum is 3.1 for CMAKE_CXX_STANDARD integration
cmake_minimum_required(VERSION 3.1)

set(CMAKE_C_COMPILER "clang")
set(CMAKE_CXX_COMPILER "clang++")

project(drampower)

set(CMAKE_CXX_STANDARD 11)

set(DRAMPOWER_XML_PATH share/xml/drampower CACHE PATH
	"Path to the directory to install XML timing/power files to.")
add_compile_options(-DXML_PATH=${CMAKE_INSTALL_PREFIX}${DRAMPOWER_XML_PATH})

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR})

find_package(XercesC REQUIRED 3.1)

add_library(lib_drampower OBJECT
	src/CommandAnalysis.cc
	src/CAHelpers.cc
	src/CmdHandlers.cc
	src/MemArchitectureSpec.cc
	src/MemCommand.cc
	src/MemoryPowerModel.cc
	src/MemorySpecification.cc
	src/MemPowerSpec.cc
	src/MemTimingSpec.cc
	src/Parameter.cc
	src/Parametrisable.cc
	src/MemBankWiseParams.cc
)

set(cli_drampower
	src/TraceParser.cc
	src/CmdScheduler.cc
	src/cli/drampower.cc
)

add_library(lib_drampowerxml OBJECT
	src/xmlparser/MemSpecParser.cc
	src/xmlparser/XMLHandler.cc
	src/xmlparser/XMLParser.cc
)

include_directories(src)

add_executable(drampower-bin
	$<TARGET_OBJECTS:lib_drampower>
	$<TARGET_OBJECTS:lib_drampowerxml>
	${cli_drampower}
)
set_target_properties(drampower-bin PROPERTIES OUTPUT_NAME drampower)
target_include_directories(drampower-bin PUBLIC ${XERCESC_INCLUDE_DIRS})
target_link_libraries(drampower-bin PUBLIC ${XercesC_LIBRARIES})
install(TARGETS drampower-bin RUNTIME DESTINATION bin)

add_library(drampower STATIC
	$<TARGET_OBJECTS:lib_drampower>
	src/libdrampower/LibDRAMPower.cc
)
install(TARGETS drampower EXPORT drampowerConfig
	ARCHIVE DESTINATION lib
	INCLUDES DESTINATION include/drampower)

add_library(drampowerxml STATIC
	$<TARGET_OBJECTS:lib_drampowerxml>
)
install(TARGETS drampowerxml EXPORT drampowerConfig
	ARCHIVE DESTINATION lib
	INCLUDES DESTINATION include/drampower)
target_include_directories(drampowerxml PUBLIC ${XERCESC_INCLUDE_DIRS})
target_link_libraries(drampowerxml PUBLIC ${XercesC_LIBRARIES})

install(DIRECTORY src/ DESTINATION include/drampower
        FILES_MATCHING PATTERN "*.h")
install(DIRECTORY src/libdrampower DESTINATION include/drampower/libdrampower
        FILES_MATCHING PATTERN "*.h")
install(DIRECTORY memspecs DESTINATION ${DRAMPOWER_XML_PATH})
install(EXPORT drampowerConfig DESTINATION lib/drampower)
