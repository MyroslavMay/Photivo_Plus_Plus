pt_add_executable(ptClear WIN32
    ../../Sources/ptClear.cpp
)

target_link_libraries(ptClear PRIVATE
    Qt${PT_QT_MAJOR_VERSION}::Widgets
)

install(TARGETS ptClear)
