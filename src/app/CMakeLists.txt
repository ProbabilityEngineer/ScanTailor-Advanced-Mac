set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

set(target_name scantailor-advanced)

set(resource_files
    resources.qrc
    dark_scheme/dark_scheme.qrc
    light_scheme/light_scheme.qrc)
list_items_prepend(resource_files "${SCANTAILOR_RESOURCES_DIR}/")

set(gui_only_sources
    RelinkablePathVisualization.cpp RelinkablePathVisualization.h
    RelinkingModel.cpp RelinkingModel.h
    RelinkingSortingModel.cpp RelinkingSortingModel.h
    RelinkingListView.cpp RelinkingListView.h
    RelinkingDialog.cpp RelinkingDialog.h
    SettingsDialog.cpp SettingsDialog.h
    FixDpiDialog.cpp FixDpiDialog.h
    LoadFilesStatusDialog.cpp LoadFilesStatusDialog.h
    ProjectCreationContext.cpp ProjectCreationContext.h
    ProjectOpeningContext.cpp ProjectOpeningContext.h
    OutOfMemoryDialog.cpp OutOfMemoryDialog.h
    ThumbnailSequence.cpp ThumbnailSequence.h
    ProjectFilesDialog.cpp ProjectFilesDialog.h
    NewOpenProjectPanel.cpp NewOpenProjectPanel.h
    SystemLoadWidget.cpp SystemLoadWidget.h
    MainWindow.cpp MainWindow.h
    main.cpp
    StatusBarPanel.cpp StatusBarPanel.h
    DefaultParamsDialog.cpp DefaultParamsDialog.h)

set(gui_only_ui_files
    AboutDialog.ui
    BatchProcessingLowerPanel.ui
    DefaultParamsDialog.ui
    FixDpiDialog.ui
    LoadFilesStatusDialog.ui
    MainWindow.ui
    NewOpenProjectPanel.ui
    OutOfMemoryDialog.ui
    ProjectFilesDialog.ui
    RelinkingDialog.ui
    RemovePagesDialog.ui
    SettingsDialog.ui
    StatusBarPanel.ui
    SystemLoadWidget.ui)

set(win32_resource_file "")
if (WIN32)
  string(REPLACE "/" "\\\\" SCANTAILOR_WIN_DIR_NATIVE "${SCANTAILOR_RESOURCES_DIR}/win32")
  configure_file("${SCANTAILOR_RESOURCES_DIR}/win32/resources.rc.in" "${CMAKE_BINARY_DIR}/resources.rc" @ONLY)
  set(rc_file "${CMAKE_BINARY_DIR}/resources.rc")
  file(GLOB win32_resources "${SCANTAILOR_RESOURCES_DIR}/win32/*.ico")
  set_source_files_properties(
      "${rc_file}" PROPERTIES
      OBJECT_DEPENDS ${win32_resources})
  if (MINGW)
    # CMake doesn't know how to process .rc files with MinGW.
    set(win32_resource_file "${CMAKE_BINARY_DIR}/win32_resources.o")
    add_custom_command(
        OUTPUT "${win32_resource_file}"
        WORKING_DIRECTORY "${SCANTAILOR_RESOURCES_DIR}/win32"
        COMMAND windres -i "${rc_file}" -o "${win32_resource_file}"
        MAIN_DEPENDENCY "${rc_file}"
        DEPENDS ${win32_resources})
  else()
    set(win32_resource_file "${rc_file}")
  endif()
endif()

set(app_bundle_type "")
if (APPLE)
  set(app_bundle_type MACOSX_BUNDLE)
endif()

add_executable(
    ${target_name} WIN32 ${app_bundle_type} ${gui_only_sources} ${gui_only_ui_files}
    ${resource_files} ${win32_resource_file})
target_link_libraries(
    ${target_name}
    PRIVATE core ${EXTRA_LIBS})
target_include_directories(
    ${target_name}
    PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}"
    "$<TARGET_PROPERTY:TIFF::TIFF,INTERFACE_INCLUDE_DIRECTORIES>")
if (APPLE)
  set(app_icon "${SCANTAILOR_RESOURCES_DIR}/macos/scantailor-advanced.icns")
  set_source_files_properties("${app_icon}" PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
  target_sources(${target_name} PRIVATE "${app_icon}")
  set_target_properties(
      ${target_name}
      PROPERTIES
      OUTPUT_NAME "ScanTailor Advanced for Mac"
      MACOSX_BUNDLE_GUI_IDENTIFIER "org.scantailor.Advanced"
      MACOSX_BUNDLE_BUNDLE_NAME "ScanTailor Advanced for Mac"
      MACOSX_BUNDLE_ICON_FILE "scantailor-advanced.icns"
      MACOSX_BUNDLE_SHORT_VERSION_STRING "${VERSION}"
      MACOSX_BUNDLE_BUNDLE_VERSION "${VERSION}")
endif()

install(TARGETS ${target_name}
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    BUNDLE DESTINATION ".")

if (Qt6_FOUND AND WIN32)
  #manage plugins with windeployqt.
  add_custom_command(
    TARGET ${target_name}
    POST_BUILD
    COMMAND ${QT_BINDIR}/windeployqt  $<TARGET_FILE:${target_name}>
    COMMENT "deploy qt files alongside binary (e.g. plugins)"
  )
  install(CODE "execute_process(COMMAND \"${QT_BINDIR}/windeployqt\" \"\$\{CMAKE_INSTALL_PREFIX\}\")")
endif()

if (Qt6_FOUND AND APPLE)
  add_custom_command(
    TARGET ${target_name}
    POST_BUILD
    COMMAND "${QT_BINDIR}/macdeployqt" "$<TARGET_BUNDLE_DIR:${target_name}>"
            -always-overwrite
            -libpath=/opt/homebrew/opt/webp/lib
            -libpath=/opt/homebrew/opt/brotli/lib
    COMMAND /usr/bin/install_name_tool -change @rpath/libsharpyuv.0.dylib @loader_path/libsharpyuv.0.dylib "$<TARGET_BUNDLE_DIR:${target_name}>/Contents/Frameworks/libwebp.7.dylib"
    COMMAND /usr/bin/install_name_tool -change @rpath/libsharpyuv.0.dylib @loader_path/libsharpyuv.0.dylib "$<TARGET_BUNDLE_DIR:${target_name}>/Contents/Frameworks/libwebpdemux.2.dylib"
    COMMAND /usr/bin/install_name_tool -change @rpath/libsharpyuv.0.dylib @loader_path/libsharpyuv.0.dylib "$<TARGET_BUNDLE_DIR:${target_name}>/Contents/Frameworks/libwebpmux.3.dylib"
    COMMAND /usr/bin/install_name_tool -change @rpath/libwebp.7.dylib @loader_path/libwebp.7.dylib "$<TARGET_BUNDLE_DIR:${target_name}>/Contents/Frameworks/libwebpdemux.2.dylib"
    COMMAND /usr/bin/install_name_tool -change @rpath/libwebp.7.dylib @loader_path/libwebp.7.dylib "$<TARGET_BUNDLE_DIR:${target_name}>/Contents/Frameworks/libwebpmux.3.dylib"
    COMMAND /usr/bin/install_name_tool -change @rpath/libbrotlicommon.1.dylib @loader_path/libbrotlicommon.1.dylib "$<TARGET_BUNDLE_DIR:${target_name}>/Contents/Frameworks/libbrotlidec.1.dylib"
    COMMAND codesign --force --deep --sign - "$<TARGET_BUNDLE_DIR:${target_name}>"
    COMMENT "Deploying and ad-hoc signing the macOS app bundle"
    VERBATIM)
endif()

translation_sources(${target_name} ${gui_only_sources} ${gui_only_ui_files})
