# libcogplatform-fdo

add_library(cogplatform-fdo MODULE
    cog-im-context-fdo-v1.c
    cog-im-context-fdo.c
    cog-platform-fdo.c
    cog-popup-menu-fdo.c
    os-compatibility.c
)
set_target_properties(cogplatform-fdo PROPERTIES
    C_STANDARD 99
    LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/modules
)
target_compile_definitions(cogplatform-fdo PRIVATE G_LOG_DOMAIN=\"Cog-FDO\")

find_package(WaylandProtocols REQUIRED)
add_wayland_protocol(cogplatform-fdo CLIENT fullscreen-shell-unstable-v1)
add_wayland_protocol(cogplatform-fdo CLIENT linux-dmabuf-unstable-v1)
add_wayland_protocol(cogplatform-fdo CLIENT presentation-time)
add_wayland_protocol(cogplatform-fdo CLIENT text-input-unstable-v1)
add_wayland_protocol(cogplatform-fdo CLIENT text-input-unstable-v3)
add_wayland_protocol(cogplatform-fdo CLIENT xdg-shell)

pkg_check_modules(Cairo IMPORTED_TARGET cairo)
pkg_check_modules(EGL IMPORTED_TARGET REQUIRED egl)
pkg_check_modules(WAYLAND IMPORTED_TARGET REQUIRED wayland-client)
pkg_check_modules(WpeFDO IMPORTED_TARGET REQUIRED wpebackend-fdo-1.0>=1.6.0)
pkg_check_modules(XkbCommon IMPORTED_TARGET REQUIRED xkbcommon)

target_link_libraries(cogplatform-fdo PRIVATE
    cogcore
    PkgConfig::Cairo
    PkgConfig::EGL
    PkgConfig::WAYLAND
    PkgConfig::WebKit
    PkgConfig::WpeFDO
)

pkg_check_modules(WAYLAND_EGL IMPORTED_TARGET wayland-egl)
if (TARGET PkgConfig::WAYLAND_EGL)
    target_link_libraries(cogplatform-fdo PRIVATE PkgConfig::WAYLAND_EGL)
endif ()

install(TARGETS cogplatform-fdo
    DESTINATION ${COG_MODULEDIR}
    COMPONENT "runtime"
)

if (COG_WESTON_DIRECT_DISPLAY)

    # Direct video display extension.
    find_path(WPEBACKEND_FDO_HAS_VIDEO_PLANE_DISPLAY_DMABUF_EXT
        NAMES wpe/extensions/video-plane-display-dmabuf.h
        PATHS ${WpeFDO_INCLUDE_DIRS}
        NO_DEFAULT_PATH
        REQUIRED
    )

    pkg_check_modules(LIBWESTON REQUIRED libweston-9-protocols)
    pkg_get_variable(LIBWESTON_PKG_DATA_DIR libweston-9-protocols "pkgdatadir")
    add_wayland_protocol(cogplatform-fdo CLIENT "${LIBWESTON_PKG_DATA_DIR}/weston-direct-display.xml")
    add_wayland_protocol(cogplatform-fdo CLIENT "${LIBWESTON_PKG_DATA_DIR}/weston-content-protection.xml")
    target_compile_definitions(cogplatform-fdo PRIVATE COG_ENABLE_WESTON_DIRECT_DISPLAY=1)
else ()
    target_compile_definitions(cogplatform-fdo PRIVATE COG_ENABLE_WESTON_DIRECT_DISPLAY=0)
endif ()

# Optional pointer support
pkg_check_modules(WaylandCursor IMPORTED_TARGET wayland-cursor)
if (TARGET PkgConfig::WaylandCursor)
    target_compile_definitions(cogplatform-fdo PRIVATE COG_USE_WAYLAND_CURSOR)
    target_link_libraries(cogplatform-fdo PRIVATE PkgConfig::WaylandCursor)
endif ()
