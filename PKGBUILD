# $Id$
# Maintainer: AndyRTR <andyrtr@archlinux.org>
# Maintainer: Jan de Groot <jgc@archlinux.org>
# Maintainer: Mike Hobbs <mhobbs@fluid.com>

pkgbase=xorg-server-bug54168
pkgname=xorg-server-bug54168
pkgver=1.16.1
pkgrel=1 # build first with 0.1 and then rebuild it after xf86-input-evdev rebuild
arch=('i686' 'x86_64')
license=('custom')
url="http://xorg.freedesktop.org"
makedepends=('pixman' 'libx11' 'mesa' 'mesa-libgl' 'xf86driproto' 'xcmiscproto' 'xtrans' 'bigreqsproto' 'randrproto' 
             'inputproto' 'fontsproto' 'videoproto' 'presentproto' 'compositeproto' 'recordproto' 'scrnsaverproto'
             'resourceproto' 'xineramaproto' 'libxkbfile' 'libxfont' 'renderproto' 'libpciaccess' 'libxv'
             'xf86dgaproto' 'libxmu' 'libxrender' 'libxi' 'dmxproto' 'libxaw' 'libdmx' 'libxtst' 'libxres'
             'xorg-xkbcomp' 'xorg-util-macros' 'xorg-font-util' 'glproto' 'dri2proto' 'libgcrypt' 'libepoxy'
             'xcb-util' 'xcb-util-image' 'xcb-util-wm' 'xcb-util-keysyms' 'dri3proto' 'libxshmfence') 
source=(${url}/releases/individual/xserver/xorg-server-${pkgver}.tar.bz2{,.sig}
        autoconfig-nvidia.patch
        autoconfig-sis.patch
        nvidia-drm-outputclass.conf
        xvfb-run
        xvfb-run.1
        06_Revert-fb-reorder-Bresenham-error-correction-to-avoi.diff)
sha256sums=('f4677c6ec9fb7b59648321737087aeb9963b60bcea50ee3773fe46be1a37e060'
            'SKIP'
            '66e25f76a7496c429e0aff4b0670f168719bb0ceaeb88c6f2272f2bf3ed21162'
            'd027776fac1f7675b0a9ee817502290b1c45f9c09b0f0a6bb058c35f92361e84'
            'af1c3d2ea5de7f6a6b5f7c60951a189a4749d1495e5462f3157ae7ac8fe1dc56'
            'ff0156309470fc1d378fd2e104338020a884295e285972cc88e250e031cc35b9'
            '2460adccd3362fefd4cdc5f1c70f332d7b578091fb9167bf88b5f91265bbd776'
            'bd40da1f2f89070286e24d39b46c1289d47bde2f5eede1a1877109870454b7a7')

prepare() {
  cd "xorg-server-${pkgver}"
  # Use unofficial imedia SiS driver for supported SiS devices
  patch -Np0 -i ../autoconfig-sis.patch
  # outputclass config file works only when nvidia-drm kernel driver is exposed
  # this means that older drivers (304xx and before) aren't currently supported
  # Use nouveau/nv/nvidia drivers for nvidia devices
  patch -Np1 -i ../autoconfig-nvidia.patch
  # Fix Freedesktop bug 54168
  patch -Np1 -i ../06_Revert-fb-reorder-Bresenham-error-correction-to-avoi.diff
}

build() {
  cd "xorg-server-${pkgver}"
  autoreconf -fi
  ./configure --prefix=/usr \
      --enable-ipv6 \
      --enable-dri \
      --enable-dmx \
      --enable-xvfb \
      --enable-xnest \
      --enable-composite \
      --enable-xcsecurity \
      --enable-xorg \
      --enable-xephyr \
      --enable-glamor \
      --enable-xwayland \
      --enable-glx-tls \
      --enable-kdrive \
      --enable-kdrive-evdev \
      --enable-kdrive-kbd \
      --enable-kdrive-mouse \
      --enable-config-udev \
      --enable-systemd-logind \
      --enable-suid-wrapper \
      --disable-install-setuid \
      --enable-record \
      --disable-xfbdev \
      --disable-xfake \
      --disable-static \
      --libexecdir=/usr/bin \
      --sysconfdir=/etc \
      --localstatedir=/var \
      --with-xkb-path=/usr/share/X11/xkb \
      --with-xkb-output=/var/lib/xkb \
      --with-fontrootdir=/usr/share/fonts
      
#      --without-dtrace \
#      --disable-linux-acpi --disable-linux-apm \

  make

  # Disable subdirs for make install rule to make splitting easier
  sed -e 's/^DMX_SUBDIRS =.*/DMX_SUBDIRS =/' \
      -e 's/^XVFB_SUBDIRS =.*/XVFB_SUBDIRS =/' \
      -e 's/^XNEST_SUBDIRS =.*/XNEST_SUBDIRS = /' \
      -e 's/^KDRIVE_SUBDIRS =.*/KDRIVE_SUBDIRS =/' \
      -e 's/^XWAYLAND_SUBDIRS =.*/XWAYLAND_SUBDIRS =/' \
      -i hw/Makefile
}

package_xorg-server-bug54168() {
  pkgdesc="Xorg X server with Debian's patch to freedesktop bug 54168"
  depends=(libepoxy libxdmcp libxfont libpciaccess libdrm pixman libgcrypt libxau xorg-server-common xf86-input-evdev libxshmfence libgl)
  # see xorg-server-*/hw/xfree86/common/xf86Module.h for ABI versions - we provide major numbers that drivers can depend on
  # and /usr/lib/pkgconfig/xorg-server.pc in xorg-server-devel pkg
  provides=('xorg-server=1.16.1' 'X-ABI-VIDEODRV_VERSION=18' 'X-ABI-XINPUT_VERSION=21' 'X-ABI-EXTENSION_VERSION=8.0' 'x-server')
  groups=('xorg')
  conflicts=('xorg-server=1.16.1' 'nvidia-utils<=331.20' 'glamor-egl')
  replaces=('glamor-egl')
  install=xorg-server.install

  cd "xorg-server-${pkgver}"
  make DESTDIR="${pkgdir}" install

  # distro specific files must be installed in /usr/share/X11/xorg.conf.d
  install -m755 -d "${pkgdir}/etc/X11/xorg.conf.d"
  install -m644 "${srcdir}/nvidia-drm-outputclass.conf" "${pkgdir}/usr/share/X11/xorg.conf.d/"

  # Needed for non-mesa drivers, libgl will restore it
  mv "${pkgdir}/usr/lib/xorg/modules/extensions/libglx.so" \
     "${pkgdir}/usr/lib/xorg/modules/extensions/libglx.xorg"

  rm -rf "${pkgdir}/var"

  rm -f "${pkgdir}/usr/share/man/man1/Xserver.1"
  rm -f "${pkgdir}/usr/lib/xorg/protocol.txt"

  install -m755 -d "${pkgdir}/usr/share/licenses/xorg-server"
  ln -sf ../xorg-server-common/COPYING "${pkgdir}/usr/share/licenses/xorg-server/COPYING"

  rm -rf "${pkgdir}/usr/lib/pkgconfig"
  rm -rf "${pkgdir}/usr/include"
  rm -rf "${pkgdir}/usr/share/aclocal"
}
