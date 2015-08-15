# Maintainer: Patryk Kowalczyk <patryk AT kowalczyk ws>
pkgname=compiz-bzr-sam
pkgver=3587
_pkgver=0.9.9
pkgrel=1
pkgdesc="Composite manager for Aiglx and Xgl, with plugins and CCSM - smspillaz experimental performance repo with buffer_age"
arch=('i686' 'x86_64')
url="https://launchpad.net/compiz"
license=('GPL' 'LGPL' 'MIT')
depends=('boost' 'xorg-server' 'libxcomposite' 'startup-notification' 'librsvg' 'dbus' 'mesa' 'libxslt' 'fuse' 'glibmm' 'libxrender' 'gmock' 'libwnck' 'pygtk' 'desktop-file-utils' 'pyrex' 'protobuf' 'metacity')
makedepends=('cmake' 'bzr' 'intltool')
optdepends=(
  'dconf: for GSettings backend support (need to rebuild this package)'
  'gnome-control-center: GNOME keybindings support (need to rebuild this package)'
  'kdebase-lib: KDE window decoration support (need to rebuild this package)' 
  'automoc: KDE window decoration support (need to rebuild this package)'
  'xorg-xprop: grab various window properties for use in window matching rules'
)
provides=()
conflicts=('compiz-bzr-sam' 'compiz-core' 'compiz-fusion-plugins-main' 'compiz-fusion-plugins-extra' 'compiz-decorator-gtk', 'compiz-decorator-kde' 'libcompizconfig' 'compizconfig-python' 'compizconfig-backend-gconf' 'compiz-bcop' 'ccsm')
replaces=('compiz-core' 'compiz-fusion-plugins-main' 'compiz-fusion-plugins-extra' 'compiz-decorator-gtk', 'compiz-decorator-kde' 'libcompizconfig' 'compizconfig-python' 'compizconfig-backend-gconf' 'compiz-bcop' 'ccsm')
install='compiz-bzr.install'

_bzrtrunk=lp:~smspillaz/compiz/experimental
#lp:~smspillaz/compiz/compiz.experimental_buffer_age_support
#lp:~smspillaz/compiz/experimental
#lp:compiz/0.9.9
_bzrmod=compiz

# GTK window decorator support
GTKWINDOWDECORATOR="On"
# Metaciy theme support for GTK window decorator
METACITY="On"
# KDE window decorator support
KDEWINDOWDECORATOR="On"
# GConf backend support
GCONF="On"
# GSettings backend support
GSETTINGS="On"

# Do some basic option validation in order to handle build failures a bit more elegantly.

if [ "${GTKWINDOWDECORATOR}" == "on" ]; then
  CHECKGCONF=`pacman -Q gconf 2>/dev/null`
  if [ ! "${CHECKGCONF}" ]; then
    msg "Currently, gconf must be installed in order to build gtk-window-decorator."
    exit 1
  fi
  if [ "${GCONF}" != "on" ]; then
    msg "Currently, GCONF must be 'on' in order to build gtk-window-decorator."
    exit 1
  fi
fi

if [ "${GSETTINGS}" == "on" ]; then
  if [ "${GCONF}" != "on" ]; then
    msg "Currently, GCONF must be 'on' in order to enable gsettings support."
    exit 1
  fi
fi

build() {
    cd "$srcdir"

    msg "Connecting to Launchpad..."
    if [[ -f "$_bzrmod/README" ]]; then
        cd "$_bzrmod" && bzr pull "$_bzrtrunk" -r "$pkgver"
    else
        bzr branch "$_bzrtrunk" "$_bzrmod" -v -r "$pkgver"
    fi
    msg "Bazaar checkout done or server timeout."

    rm -rf "$srcdir/$_bzrmod-build"
    msg "Creating build directory..."
    cp -r "$srcdir/$_bzrmod" "$srcdir/$_bzrmod-build"
    cd "$srcdir/$_bzrmod-build"

    export PYTHON="/usr/bin/python2"
    find -type f \( -name 'CMakeLists.txt' -or -name '*.cmake' \) -exec sed -e 's/COMMAND python/COMMAND python2/g' -i {} \;
    find compizconfig/ccsm -type f -exec sed -e 's|^#!.*python|#!/usr/bin/env python2|g' -i {} \;

    msg "Running cmake..." 

    mkdir build; cd build
    cmake .. \
        -DCMAKE_INSTALL_PREFIX="/usr" \
        -DPYTHON_INCLUDE_DIR=/usr/include/python2.7 \
        -DPYTHON_LIBRARY=/usr/lib/libpython2.7.so \
        -DCOMPIZ_DISABLE_SCHEMAS_INSTALL=On \
        -DCOMPIZ_BUILD_WITH_RPATH=Off \
        -DCOMPIZ_PACKAGING_ENABLED=On \
        -DBUILD_GTK="${GTKWINDOWDECORATOR}" \
        -DBUILD_METACITY="${METACITY}" \
        -DBUILD_KDE4="${KDEWINDOWDECORATOR}" \
        -DUSE_GCONF="${GCONF}" \
        -DUSE_GSETTINGS="${GSETTINGS}" \
        -DCOMPIZ_BUILD_TESTING=Off \
        -DCOMPIZ_DEFAULT_PLUGINS="composite,opengl,decor,resize,place,move"
    make
}

package() {
    cd "$srcdir/$_bzrmod-build/build"
    make DESTDIR="${pkgdir}" install

    # Stupid findcompiz_install needs COMPIZ_DESTDIR and install needs DESTDIR
    # make findcompiz_install
    CMAKE_DIR=$(cmake --system-information | grep '^CMAKE_ROOT' | awk -F\" '{print $2}')
    install -dm755 "${pkgdir}${CMAKE_DIR}/Modules/"
    install -m644 ../cmake/FindCompiz.cmake "${pkgdir}${CMAKE_DIR}/Modules/"	

    # Add documentation
    install -dm755 "${pkgdir}/usr/share/doc/compiz/"
    install ../{AUTHORS,NEWS,README} "${pkgdir}/usr/share/doc/compiz/"

    # Amend XDG .desktop file to load the compizconfig plugin with compiz
    sed -i 's/Exec\=compiz/Exec\=compiz ccp/' "${pkgdir}/usr/share/applications/compiz.desktop"

    # Merge the gconf schema files
    if [ -d "${pkgdir}/usr/share/gconf/schemas" ]; then    
        gconf-merge-schema "${pkgdir}/usr/share/gconf/schemas/compiz.schemas.in" "{$pkgdir}"/usr/share/gconf/schemas/*.schemas
        sed -i '/<schemalist\/>/d' "${pkgdir}/usr/share/gconf/schemas/compiz.schemas.in"
        rm -f "${pkgdir}"/usr/share/gconf/schemas/*.schemas
        mv "${pkgdir}/usr/share/gconf/schemas/compiz.schemas.in" "${pkgdir}/usr/share/gconf/schemas/compiz.schemas"
    fi

    # Add the pesky gsettings schema files manually
    if ls generated | grep -qm1 .gschema.xml; then
        install -dm755 "${pkgdir}/usr/share/glib-2.0/schemas/" 
        install -m644 generated/*.gschema.xml "${pkgdir}/usr/share/glib-2.0/schemas/" 
    fi  
}

