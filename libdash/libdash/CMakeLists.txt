cmake_minimum_required(VERSION 2.8)

unset(REQUIRED_LIBRARIES)
unset(REQUIRED_INCLUDE_DIRS)
unset(REQUIRED_FILES)

get_filename_component(REQUIRED_INCLUDE_DIRS "./include" REALPATH)

if(MSVC)

    list(APPEND REQUIRED_LIBRARIES wsock32 ws2_32)

    macro(libdash_add_external _name _lib_path)
        get_filename_component(_target_path "../${_name}" REALPATH)
        list(APPEND REQUIRED_LIBRARIES     "${_target_path}/${_lib_path}")
        list(APPEND REQUIRED_INCLUDE_DIRS  "${_target_path}/include")

        file(GLOB_RECURSE _dlls "${_target_path}/*.dll")
        if(_dlls)
            list(APPEND REQUIRED_FILES  "${_dlls}")
        endif()
    endmacro(libdash_add_external)

    libdash_add_external(iconv   lib/iconv.lib)
    libdash_add_external(libxml2 lib/libxml2.lib)
    libdash_add_external(libcurl lib/libcurl_imp.lib)
    # libdash_add_external(sdl     lib/x86/SDL.lib)
    libdash_add_external(zlib    lib/zlib.lib)

else(MSVC)

    find_package(LibXml2 REQUIRED)
    find_package(ZLIB REQUIRED)
    find_package(CURL REQUIRED)

    set(REQUIRED_LIBRARIES ${CURL_LIBRARIES}
                           ${ZLIB_LIBRARIES}
                           ${LIBXML2_LIBRARIES})
    list(APPEND REQUIRED_INCLUDE_DIRS ${CURL_INCLUDE_DIRS}
                                      ${ZLIB_INCLUDE_DIRS}
                                      ${LIBXML2_INCLUDE_DIR})

endif(MSVC)

list(REMOVE_DUPLICATES REQUIRED_LIBRARIES)
list(REMOVE_DUPLICATES REQUIRED_INCLUDE_DIRS)

message(STATUS "REQUIRED_LIBRARIES:     ${REQUIRED_LIBRARIES}")
message(STATUS "REQUIRED_INCLUDE_DIRS:  ${REQUIRED_INCLUDE_DIRS}")
message(STATUS "REQUIRED_FILES:         ${REQUIRED_FILES}")

include_directories(${REQUIRED_INCLUDE_DIRS})

file(GLOB_RECURSE libdash_source *.cpp)

add_library(dash SHARED ${libdash_source})

target_link_libraries(dash ${REQUIRED_LIBRARIES})

if (REQUIRED_FILES)
    foreach(_required_file ${REQUIRED_FILES})
        add_custom_command(TARGET dash
                           POST_BUILD
                           COMMAND ${CMAKE_COMMAND} -E copy_if_different
                                                    "${_required_file}"
                                                    $<TARGET_FILE_DIR:dash>)
    endforeach()
endif()
