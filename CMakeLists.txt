cmake_minimum_required(VERSION 2.8.12)
project(sqfvm)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_MACOSX_RPATH 1)

# Enable multithreaded compilation in MSVC
add_compile_options($<$<CXX_COMPILER_ID:MSVC>:/MP>)

# Add the filesystem library if we are building on Clang or GCC
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    set(ST_CXXFS_LIBS stdc++fs)
endif()

set(ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/output)
set(LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/output)
set(RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/output)

include_directories("${PROJECT_SOURCE_DIR}/include/json/include")
include_directories("${PROJECT_SOURCE_DIR}/include/tclap-1.2.2/include")

file(GLOB vm_src 
    "${PROJECT_SOURCE_DIR}/src/*.h" "${PROJECT_SOURCE_DIR}/src/*.cpp" "${PROJECT_SOURCE_DIR}/src/*.c"
    "${PROJECT_SOURCE_DIR}/src/commands/*.h" "${PROJECT_SOURCE_DIR}/src/commands/*.cpp" "${PROJECT_SOURCE_DIR}/src/commands/*.c"
    "${CMAKE_CURRENT_BINARY_DIR}/git_sha1.cpp"
)

# Get the local git revision hash and put it into a header we can use
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/")
include(GetGitRevisionDescription)
get_git_head_revision(GIT_REFSPEC GIT_SHA1)

configure_file("${PROJECT_SOURCE_DIR}/cmake/git_sha1.cpp.in" "${CMAKE_CURRENT_BINARY_DIR}/git_sha1.cpp" @ONLY)
list(APPEND vm_src "${CMAKE_CURRENT_BINARY_DIR}/git_sha1.cpp")


find_package(Threads)


option(SQFVM_BUILD_EXECUTABLE "BUILD EXECUTABLE" ON)
option(SQFVM_BUILD_LIBRARY "BUILD LIBRARY" ON)
option(SQFVM_BUILD_STATIC_LIBRARY "BUILD STATIC LIBRARY" ON)
option(SQFVM_BUILD_STATIC_LIBRARY_NC "BUILD STATIC LIBRARY WITHOUT COMMANDS" ON)


if (SQFVM_BUILD_EXECUTABLE)
    add_executable(sqfvm ${vm_src})
    target_link_libraries(sqfvm ${CMAKE_DL_LIBS} ${CMAKE_THREAD_LIBS_INIT} ${ST_CXXFS_LIBS})
    target_include_directories(sqfvm PUBLIC ${PROJECT_SOURCE_DIR}/src)
endif ()


if (SQFVM_BUILD_LIBRARY)
    add_library(libsqfvm SHARED ${vm_src})
    target_include_directories(libsqfvm PUBLIC ${PROJECT_SOURCE_DIR}/src)
    SET_TARGET_PROPERTIES(libsqfvm PROPERTIES PREFIX "")
endif ()

if (SQFVM_BUILD_STATIC_LIBRARY)

    list(REMOVE_ITEM vm_src "${PROJECT_SOURCE_DIR}/src/Entry.cpp")
    list(REMOVE_ITEM vm_src "${PROJECT_SOURCE_DIR}/src/dllexports.cpp")

    add_library(slibsqfvm STATIC ${vm_src})
    target_include_directories(slibsqfvm PUBLIC ${PROJECT_SOURCE_DIR}/src)
    SET_TARGET_PROPERTIES(slibsqfvm PROPERTIES PREFIX "")
endif ()

if (SQFVM_BUILD_STATIC_LIBRARY_NC)

	file(GLOB vm_src_nc
		"${PROJECT_SOURCE_DIR}/src/*.h" "${PROJECT_SOURCE_DIR}/src/*.cpp" "${PROJECT_SOURCE_DIR}/src/*.c"
	)

    list(REMOVE_ITEM vm_src_nc "${PROJECT_SOURCE_DIR}/src/Entry.cpp")
    list(REMOVE_ITEM vm_src_nc "${PROJECT_SOURCE_DIR}/src/dllexports.cpp")

    add_library(slibsqfvm_nc STATIC ${vm_src_nc})
    target_include_directories(slibsqfvm_nc PUBLIC ${PROJECT_SOURCE_DIR}/src)
    SET_TARGET_PROPERTIES(slibsqfvm_nc PROPERTIES PREFIX "")
endif ()