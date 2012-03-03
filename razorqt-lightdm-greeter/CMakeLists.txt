set(PROJECT razor-lightdm-greeter)
project($PROJECT)

include(FindPkgConfig)
pkg_check_modules(LIGHTDM_QT liblightdm-qt-1)


if(NOT LIGHTDM_QT_INCLUDE_DIRS)
    message(STATUS "Qt bindings for liblightdm not found. Razor-lightdm-greeter won't be built")
    message(STATUS "Hint: On debian or debian-derived distributions you could probably do: sudo apt-get liblightdm-qt-1-dev")
    message(STATUS "      On suse or opensuse distribution: lightdm-qt-devel")
elseif(NOT LIGHTDM_QT_LIBRARIES)
    message(STATUS "lightdm-qt library not found. Razor-lightdm-greeter won't be built")
    message(STATUS "Hint: On debian or debian-derived distributions the package would be something like: liblightdm-qt-1-0")
    message(STATUS "      On suse or opensuse distribution: liblightdm-qt-1-0")
else()

    ############### Patch #############################################################
    # liblightdm-qt-1.pc (pkg-config file) erroneously reports includedir as
    # /usr/include/lightdm-qt-0 and the library as lightdm-qt-0. So we have to patch
    # Bug filed: https://bugs.launchpad.net/ubuntu/+source/lightdm/+bug/943914

    foreach(INCLUDEDIR ${LIGHTDM_QT_INCLUDE_DIRS})
        string(REPLACE "-qt-0" "-qt-1" INCLUDEDIR ${INCLUDEDIR})
        list(APPEND TMP1 ${INCLUDEDIR})
    endforeach()
    set(LIGHTDM_QT_INCLUDE_DIRS ${TMP1})

    foreach(LIBRARY ${LIGHTDM_QT_LIBRARIES})
        string(REPLACE "-qt-0" "-qt-1" LIBRARY ${LIBRARY})
        list(APPEND TMP2 ${LIBRARY})
    endforeach()
    set(LIGHTDM_QT_LIBRARIES ${TMP2})
    ############## End patch #########################################################

    include_directories (
            ${CMAKE_CURRENT_SOURCE_DIR}
            ${CMAKE_CURRENT_BINARY_DIR}
            ${CMAKE_SOURCE_DIR}/libraries/
            ${CMAKE_SOURCE_DIR}/libraries/razorqt
            ${LIGHTDM_QT_INCLUDE_DIRS}
            )

    set ( razor-lightdm-greeter_SRCS
          main.cpp
          mainwindow.cpp
          loginform.cpp
        )

    set ( razor-lightdm-greeter_UIS
          loginform.ui
        )
    QT4_WRAP_UI(UIS ${razor-lightdm-greeter_UIS})

    set ( razor-lightdm-greeter_RSCS )
    QT4_ADD_RESOURCES(RSCS ${razor-lightdm-greeter_RSCS})

    set ( razor-lightdm-greeter_MOCS
          mainwindow.h
          loginform.h
        )
    QT4_WRAP_CPP(MOCS ${razor-lightdm-greeter_MOCS})

    if(CMAKE_BUILD_TYPE MATCHES [Dd]ebug)
        configure_file(test/test.sh test/test.sh)
        configure_file(test/lightdm-razor-greeter.desktop test/lightdm-razor-greeter.desktop)
        configure_file(test/lightdm.conf test/lightdm.conf)
    endif()

    add_executable ( razor-lightdm-greeter ${razor-lightdm-greeter_SRCS} ${UIS} ${RSCS} ${TRS} ${MOCS} )
    target_link_libraries ( razor-lightdm-greeter  razorqt ${QT_QTCORE_LIBRARY} ${QT_QTGUI_LIBRARY} ${QT_QTCORE_LIBRARY} ${QT_QTGUI_LIBRARY} ${QT_QTDBUS_LIBRARY} ${LIGHTDM_QT_LIBRARIES})

    file(GLOB DESKTOP_FILES resources/*.desktop)

    install(TARGETS ${PROJECT} RUNTIME DESTINATION bin)
    install(FILES   ${DESKTOP_FILES}   DESTINATION /usr/share/xgreeters)
endif()