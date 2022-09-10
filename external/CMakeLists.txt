set(BUILD_TESTING OFF CACHE BOOL "" FORCE)
add_subdirectory(nifly)

# DirectX packages for unix builds (included as submodules to avoid vcpkg dependency)
if(UNIX)
    function(export_package NAME)
        export(TARGETS ${NAME} NAMESPACE Microsoft:: FILE ${CMAKE_INSTALL_DATAROOTDIR}/${NAME}/cmake/${NAME}-targets.cmake)
        export(PACKAGE ${NAME})
        string(TOLOWER ${NAME} PACKAGE)
        file(WRITE ${CMAKE_BINARY_DIR}/${PACKAGE}/${PACKAGE}-config.cmake "")
        set(${PACKAGE}_DIR ${CMAKE_BINARY_DIR}/${PACKAGE} CACHE PATH "")
    endfunction()

    set(DXHEADERS_INSTALL ON CACHE BOOL "" FORCE)
    add_subdirectory(DirectX-Headers)
    export_package(DirectX-Headers)

    add_subdirectory(DirectXMath)
    add_library(Microsoft::DirectXMath ALIAS DirectXMath)
    export_package(DirectXMath)

    # DirectXMath doesn't ship sal.h despite it's "supporting" unix...
    add_library(unix-compat INTERFACE IMPORTED)
    target_include_directories(unix-compat SYSTEM INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/compat/unix")
    target_link_libraries(DirectXMath INTERFACE unix-compat)

    # DirectXTex uses malloc.h despite it being non-standard for 33+ years...
    if(APPLE)
        add_library(apple-compat INTERFACE IMPORTED)
        target_include_directories(apple-compat SYSTEM INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/compat/apple")
        target_link_libraries(unix-compat INTERFACE apple-compat)
    endif()
endif()

# Unfortunately DirectXTex has a lot of options we don't care about
set(BUILD_TOOLS OFF CACHE BOOL "" FORCE)
set(BUILD_SAMPLE OFF CACHE BOOL "" FORCE)
set(BUILD_SAMPLE OFF CACHE BOOL "" FORCE)
set(BUILD_DX11 OFF CACHE BOOL "" FORCE)
set(BUILD_DX12 OFF CACHE BOOL "" FORCE)
add_subdirectory(DirectXTex)
