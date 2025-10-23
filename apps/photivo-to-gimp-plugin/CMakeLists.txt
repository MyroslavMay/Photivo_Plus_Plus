if ("${gimp_VERSION}" VERSION_GREATER_EQUAL "2.8")
    message(
        FATAL_ERROR
        "Detected GIMP ${gimp_VERSION}. The old Photivo-to-GIMP plugin can only be built with "
        "GIMP 2.6 or 2.8. For newer GIMPs it is not needed any more."
    )
endif()

pt_add_executable(ptGimp WIN32
    ../../Sources/ptGimp.cpp
    ../../Sources/ptCalloc.cpp
    ../../Sources/ptError.cpp
)

target_link_libraries(ptGimp PRIVATE
    Qt${PT_QT_MAJOR_VERSION}::Core
    PkgConfig::gimp
    PkgConfig::gtk2
)

install(TARGETS ptGimp DESTINATION "${CMAKE_INSTALL_LIBDIR}/gimp/2.0/plug-ins")
