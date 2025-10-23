pt_add_executable(ptCreateAdobeProfiles
    ../../Sources/ptCreateAdobeProfiles.cpp
    ../../Sources/ptColorProfiles.h
    ../../Sources/ptConstants.h
    ../../Sources/tools/finally.h
)

target_link_libraries(ptCreateAdobeProfiles PRIVATE
    Qt${PT_QT_MAJOR_VERSION}::Core
    PkgConfig::lcms2
    PkgConfig::lensfun
    PkgConfig::libraw
)

install(TARGETS ptCreateAdobeProfiles)
