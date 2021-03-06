ExternalProject_Add(binutils
    URL "http://ftp.gnu.org/gnu/binutils/binutils-2.25.tar.bz2"
    URL_HASH SHA256=22defc65cfa3ef2a3395faaea75d6331c6e62ea5dfacfed3e2ec17b08c882923
    CONFIGURE_COMMAND <SOURCE_DIR>/configure
        --target=${TARGET_ARCH}
        --prefix=${CMAKE_INSTALL_PREFIX}
        --with-sysroot=${CMAKE_INSTALL_PREFIX}
        --disable-multilib
        --disable-nls
        --disable-shared
    BUILD_COMMAND make -j${MAKEJOBS}
    INSTALL_COMMAND make install
    LOG_DOWNLOAD 1 LOG_CONFIGURE 1 LOG_BUILD 1 LOG_INSTALL 1
)

find_program(PKGCONFIG NAMES pkg-config)

ExternalProject_Add_Step(binutils basedirs
    DEPENDEES download
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_INSTALL_PREFIX}
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_INSTALL_PREFIX}/bin
    COMMAND ${CMAKE_COMMAND} -E create_symlink ${PKGCONFIG} ${CMAKE_INSTALL_PREFIX}/bin/${TARGET_ARCH}-pkg-config
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_INSTALL_PREFIX}/${TARGET_ARCH}
    COMMAND ${CMAKE_COMMAND} -E create_symlink ${CMAKE_INSTALL_PREFIX}/${TARGET_ARCH} ${CMAKE_INSTALL_PREFIX}/mingw
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_INSTALL_PREFIX}/${TARGET_ARCH}/lib
    COMMAND ${CMAKE_COMMAND} -E create_symlink ${CMAKE_INSTALL_PREFIX}/${TARGET_ARCH}/lib ${CMAKE_INSTALL_PREFIX}/${TARGET_ARCH}/lib64
    COMMENT "Setting up target directories and symlinks"
)

ExternalProject_Add(mingw-w64-headers
    PREFIX mingw-w64-prefix
    STAMP_DIR mingw-w64-prefix/src/mingw-w64-stamp
    SOURCE_DIR mingw-w64-prefix/src/mingw-w64
    BINARY_DIR mingw-w64-prefix/src/mingw-w64-headers-build
    DEPENDS binutils
    URL "http://downloads.sourceforge.net/project/mingw-w64/mingw-w64/mingw-w64-release/mingw-w64-v4.0.2.tar.bz2"
    URL_HASH SHA256=3e9050a8c6689ef8a0cfafa40a7653e8c347cf93c105d547239c573afe7b8952
    PATCH_COMMAND patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/mingw-w64-1.patch
    CONFIGURE_COMMAND <SOURCE_DIR>/mingw-w64-headers/configure
        --host=${TARGET_ARCH}
        --prefix=${MINGW_INSTALL_PREFIX}
        --enable-sdk=all
    BUILD_COMMAND ""
    INSTALL_COMMAND make install
    LOG_DOWNLOAD 1 LOG_UPDATE 1 LOG_CONFIGURE 1 LOG_BUILD 1 LOG_INSTALL 1
)

ExternalProject_Add(gcc-base
    PREFIX gcc-prefix
    STAMP_DIR gcc-prefix/src/gcc-stamp
    SOURCE_DIR gcc-prefix/src/gcc
    BINARY_DIR gcc-prefix/src/gcc-build
    DEPENDS mingw-w64-headers
    URL "ftp://ftp.gwdg.de/pub/misc/gcc/releases/gcc-5.1.0/gcc-5.1.0.tar.bz2"
    URL_HASH SHA512=30f6a94d3adb25bc51fcaddf32a6c41429f569eeb9ed64330445b1296f99998fbfa03277b375be4f7b2b80f519910ef88e4149db7cd9031e2c61a49f501bde94
    CONFIGURE_COMMAND <SOURCE_DIR>/configure
        --target=${TARGET_ARCH}
        --prefix=${CMAKE_INSTALL_PREFIX}
        --libdir=${CMAKE_INSTALL_PREFIX}/lib
        --with-sysroot=${CMAKE_INSTALL_PREFIX}
        --disable-multilib
        --enable-languages=c,c++
        --disable-nls
        --disable-shared
        --disable-win32-registry
    BUILD_COMMAND make -j${MAKEJOBS} all-gcc
    INSTALL_COMMAND make install-gcc
    LOG_DOWNLOAD 1 LOG_CONFIGURE 1 LOG_BUILD 1 LOG_INSTALL 1
)

ExternalProject_Add(mingw-w64-crt
    PREFIX mingw-w64-prefix
    SOURCE_DIR mingw-w64-prefix/src
    DEPENDS gcc-base
    CONFIGURE_COMMAND ${EXEC} <SOURCE_DIR>/mingw-w64/mingw-w64-crt/configure
        --host=${TARGET_ARCH}
        --prefix=${MINGW_INSTALL_PREFIX}
        --with-sysroot=${CMAKE_INSTALL_PREFIX}
    BUILD_COMMAND ${MAKE}
    INSTALL_COMMAND ${MAKE} install
    LOG_DOWNLOAD 1 LOG_UPDATE 1 LOG_CONFIGURE 1 LOG_BUILD 1 LOG_INSTALL 1
)

ExternalProject_Add(gcc
    DEPENDS mingw-w64-crt
    SOURCE_DIR gcc-prefix/src
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ${MAKE}
    INSTALL_COMMAND ${MAKE} install
    LOG_BUILD 1 LOG_INSTALL 1
)

ExternalProject_Add(winpthreads
    PREFIX mingw-w64-prefix
    SOURCE_DIR mingw-w64-prefix/src
    DEPENDS gcc
    CONFIGURE_COMMAND ${EXEC} <SOURCE_DIR>/mingw-w64/mingw-w64-libraries/winpthreads/configure
        --host=${TARGET_ARCH}
        --prefix=${MINGW_INSTALL_PREFIX}
        --disable-shared
        --enable-static
    BUILD_COMMAND ${MAKE}
    INSTALL_COMMAND ${MAKE} install
    LOG_CONFIGURE 1 LOG_BUILD 1 LOG_INSTALL 1
)
