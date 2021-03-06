
# Functions to get target's link libs and include dirs.
function(GetLinkLibraries ret target)
    if(TARGET ${target})
        get_property(LIBS TARGET ${target} PROPERTY INTERFACE_LINK_LIBRARIES)
        set(RESULT "${target}")
        if(LIBS)
            foreach(LIB ${LIBS})
                GetLinkLibraries(SUB_LIBS ${LIB})
                if(SUB_LIBS)
                    foreach(SUB_LIB ${SUB_LIBS})
                        if(NOT ${SUB_LIB} IN_LIST RESULT)
                            list(APPEND RESULT ${SUB_LIBS})
                        endif()
                    endforeach()
                endif()
            endforeach()
        endif()
    endif()
    set(${ret} ${RESULT} PARENT_SCOPE)
endfunction()
function(GetIncludeDirectories ret target)
    GetLinkLibraries(LIBS ${target})
    set(RESULT "")
    foreach(LIB ${LIBS})
        get_property(INCLUDES TARGET ${LIB} PROPERTY INTERFACE_INCLUDE_DIRECTORIES)
        if(INCLUDES)
            foreach(INCL ${INCLUDES})
                if(NOT ${INCL} IN_LIST RESULT)
                    list(APPEND RESULT ${INCL})
                endif()
            endforeach()
        endif()
    endforeach()
    set(${ret} ${RESULT} PARENT_SCOPE)
endfunction()

# fileformat-headers target can be linked with targets to get fileformat's
# includes without actually linking fileformat's libraries.
GetIncludeDirectories(FILEFORMAT_HEADERS fileformat)
add_library(fileformat-headers INTERFACE)
target_include_directories(fileformat-headers INTERFACE ${FILEFORMAT_HEADERS})
add_dependencies(fileformat-headers fileformat)

# Subdirectories.
add_subdirectory(mpress)
add_subdirectory(upx)
