# Builds the qtui module

set(SOURCES
    aboutdlg.cpp
    awaylogfilter.cpp
    awaylogview.cpp
    bufferwidget.cpp
    channellistdlg.cpp
    chatitem.cpp
    chatline.cpp
    chatlinemodel.cpp
    chatlinemodelitem.cpp
    chatmonitorfilter.cpp
    chatmonitorview.cpp
    chatscene.cpp
    chatview.cpp
    chatviewsearchbar.cpp
    chatviewsearchcontroller.cpp
    chatviewsettings.cpp
    columnhandleitem.cpp
    coreconfigwizard.cpp
    coreconnectdlg.cpp
    coreconnectionstatuswidget.cpp
    coreinfodlg.cpp
    debugbufferviewoverlay.cpp
    debugconsole.cpp
    debuglogwidget.cpp
    debugmessagemodelfilter.cpp
    inputwidget.cpp
    ircconnectionwizard.cpp
    legacysystemtray.cpp
    mainpage.cpp
    mainwin.cpp
    markerlineitem.cpp
    msgprocessorstatuswidget.cpp
    nicklistwidget.cpp
    qtui.cpp
    qtuiapplication.cpp
    qtuimessageprocessor.cpp
    qtuisettings.cpp
    qtuistyle.cpp
    receivefiledlg.cpp
    settingsdlg.cpp
    settingspagedlg.cpp
    simplenetworkeditor.cpp
    systemtray.cpp
    systraynotificationbackend.cpp
    taskbarnotificationbackend.cpp
    titlesetter.cpp
    topicwidget.cpp
    verticaldock.cpp
    webpreviewitem.cpp
)

set(FORMS
    aboutdlg.ui
    bufferviewwidget.ui
    bufferwidget.ui
    channellistdlg.ui
    chatviewsearchbar.ui
    coreconfigwizardintropage.ui
    coreconfigwizardadminuserpage.ui
    coreconfigwizardstorageselectionpage.ui
    coreconfigwizardsyncpage.ui
    coreconnectauthdlg.ui
    coreconnectionstatuswidget.ui
    coreinfodlg.ui
    debugbufferviewoverlay.ui
    debugconsole.ui
    debuglogwidget.ui
    inputwidget.ui
    msgprocessorstatuswidget.ui
    nicklistwidget.ui
    settingsdlg.ui
    settingspagedlg.ui
    simplenetworkeditor.ui
    receivefiledlg.ui
    topicwidget.ui
)

set(LIBS )
set(QT_MODULES )

if (KDE4_FOUND)
    add_definitions(-DHAVE_KDE ${KDE4_DEFINITIONS})
    include_directories(${KDE4_INCLUDES})
    list(APPEND SOURCES knotificationbackend.cpp)
    list(APPEND LIBS ${KDE4_KDECORE_LIBS} ${KDE4_KDEUI_LIBRARY} ${KDE4_KNOTIFYCONFIG_LIBRARY})
endif()

if (LIBSNORE_FOUND)
    add_definitions(-DHAVE_SNORENOTIFY)
    include_directories(${LIBSNORE_INCLUDE_DIRS})
    list(APPEND SOURCES snorenotificationbackend.cpp)
    list(APPEND FORMS   snorentificationconfigwidget.ui)
    list(APPEND LIBS ${LIBSNORE_LIBRARIES})
endif()

if (PHONON_FOUND OR Phonon4Qt5_FOUND)
    add_definitions(-DHAVE_PHONON)
    include_directories(${PHONON_INCLUDES})
    set(SOURCES ${SOURCES} phononnotificationbackend.cpp)
    set(FORMS ${FORMS}     phononnotificationconfigwidget.ui)
    list(APPEND LIBS ${PHONON_LIBS})  # PHONON_LIBRARIES only exists in config mode, it's not set by the legacy FindPhonon.cmake
endif()

if (QT_QTDBUS_FOUND OR Qt5DBus_FOUND)
    add_definitions(-DHAVE_DBUS)
    list(APPEND QT_MODULES DBus)
    if (dbusmenu-qt_FOUND OR dbusmenu-qt5_FOUND)
        add_definitions(-DHAVE_DBUSMENU)
        include_directories(${dbusmenu-qt_INCLUDE_DIRS} ${dbusmenu-qt5_INCLUDE_DIRS})
        if (dbusmenu-qt_FOUND)
            list(APPEND LIBS dbusmenu-qt)
        else()
            list(APPEND LIBS dbusmenu-qt5)
        endif()
    endif()

    list(APPEND SOURCES statusnotifieritem.cpp statusnotifieritemdbus.cpp dockmanagernotificationbackend.cpp)
    qt_add_dbus_interface(SOURCES ../../interfaces/org.kde.StatusNotifierWatcher.xml statusnotifierwatcher)
    qt_add_dbus_interface(SOURCES ../../interfaces/org.freedesktop.Notifications.xml notificationsclient)
    qt_add_dbus_adaptor  (SOURCES ../../interfaces/org.kde.StatusNotifierItem.xml statusnotifieritemdbus.h StatusNotifierItemDBus)
endif()

if (QT_QTWEBKIT_FOUND OR Qt5WebKitWidgets_FOUND)
    add_definitions(-DHAVE_WEBKIT)
    list(APPEND QT_MODULES WebKit XmlPatterns)
    if (USE_QT5)
        list(APPEND QT_MODULES WebKitWidgets)
    endif()
endif()

if(HAVE_SSL)
  set(SOURCES ${SOURCES} sslinfodlg.cpp)
  set(FORMS ${FORMS} sslinfodlg.ui)
endif(HAVE_SSL)

if (INDICATEQT_FOUND)
    add_definitions(-DHAVE_INDICATEQT)
    include_directories(${INDICATEQT_INCLUDE_DIRS})
    list(APPEND SOURCES indicatornotificationbackend.cpp)
    list(APPEND FORMS   indicatornotificationconfigwidget.ui)
    list(APPEND LIBS ${INDICATEQT_LIBRARIES})
endif()

if (WITH_NOTIFICATION_CENTER)
    add_definitions(-DHAVE_NOTIFICATION_CENTER)
    list(APPEND SOURCES osxnotificationbackend.mm)
    list(APPEND LIBS "/System/Library/Frameworks/Foundation.framework")
endif()

foreach(FORM ${FORMS})
  set(FORMPATH ${FORMPATH} ui/${FORM})
endforeach(FORM ${FORMS})

# handle settingspages
include(settingspages/settingspages.cmake)
foreach(SP ${SETTINGSPAGES})
  set(SPSRC ${SPSRC} settingspages/${SP}settingspage.cpp)
  set(SPFRM ${SPFRM} settingspages/${SP}settingspage.ui)
endforeach(SP ${SETTINGSPAGES})
foreach(SRC ${SP_SOURCES})
  set(SPSRC ${SPSRC} settingspages/${SRC})
endforeach(SRC ${SP_SOURCES})
foreach(FRM ${SP_FORMS})
  set(SPFRM ${SPFRM} settingspages/${FRM})
endforeach(FRM ${SP_FORMS})

include_directories(${CMAKE_SOURCE_DIR}/src/common
                    ${CMAKE_SOURCE_DIR}/src/client
                    ${CMAKE_SOURCE_DIR}/src/qtui
                    ${CMAKE_SOURCE_DIR}/src/qtui/settingspages
                    ${CMAKE_SOURCE_DIR}/src/uisupport

)

qt_wrap_ui(UI ${FORMPATH} ${SPFRM})

if (USE_QT5)
    list(APPEND QT_MODULES Widgets)
endif()


add_library(mod_qtui STATIC ${SOURCES} ${SPSRC} ${UI})
qt_use_modules(mod_qtui Core Gui Network ${QT_MODULES})

if (LIBS)
    target_link_libraries(mod_qtui ${LIBS})
endif()

add_dependencies(mod_qtui mod_common mod_client mod_uisupport)
