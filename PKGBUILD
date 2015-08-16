# $Id: PKGBUILD 80217 2012-11-17 21:52:58Z lcarlier $
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>
# Maintainer: Fantix King <fantix.king at gmail.com>

pkgbase=libx32-mesa
pkgname=libx32-mesa
true && pkgname=('libx32-libglapi' 'libx32-libgl' 'libx32-mesa' 'libx32-osmesa' 'libx32-libgles' 'libx32-ati-dri' 'libx32-intel-dri'
         'libx32-nouveau-dri') # libx32-libgbm needs udev

#_git=true
#_gitdate=20121005
_git=false

if [ "${_git}" = "true" ]; then
    pkgver=8.99.git_$_gitdate
  else
    pkgver=9.0.1
fi

pkgrel=1.1
arch=('x86_64')
makedepends=('glproto>=1.4.16' 'libx32-libdrm>=2.4.39' 'libx32-libxxf86vm>=1.1.2' 'libx32-libxdamage>=1.1.3' 'libx32-expat>=2.1.0'
             'libx32-libx11>=1.5.0' 'libx32-libxt>=1.1.3' 'libx32-gcc-libs>=4.7.1-6' 'dri2proto>=2.8' 'python2' 'libxml2'
             'gcc-multilib-x32' 'imake' 'libx32-llvm')
url="http://mesa3d.sourceforge.net"
license=('custom')
options=('!libtool')
#source=(git_fixes.diff)
source=()
if [ "${_git}" = "true" ]; then
  # mesa git shot from 9.0 branch - see for state: http://cgit.freedesktop.org/mesa/mesa/log/?h=9.0
  #source=(${source[@]} 'ftp://ftp.archlinux.org/other/mesa/mesa-41d14eaf193c6b1eb87fe1998808a887f1c6c698.tar.gz')
  source=(${source[@]} "MesaLib-git${_gitdate}.zip"::"http://cgit.freedesktop.org/mesa/mesa/snapshot/mesa-542f6feda9bf18267dbd337943a5e871400d425a.tar.gz")
else
  source=(${source[@]} "ftp://ftp.freedesktop.org/pub/mesa/${pkgver}/MesaLib-${pkgver}.tar.bz2")
fi
md5sums=('97d6554c05ea7449398afe3a0ede7018')

build() {
  export CC="gcc -mx32"
  export CXX="g++ -mx32"
  export PKG_CONFIG_PATH="/usr/libx32/pkgconfig"
  # for our llvm-config for x32 ABI
  export LLVM_CONFIG=/usr/bin/llvm-configx32

  # fix segfault with gfx cards > Ati R700
  # export CFLAGS="${CFLAGS} -O1"
  # export CXXFLAGS="${CXXFLAGS} -O1"

  cd ${srcdir}/?esa-*

  COMMONOPTS="--prefix=/usr \
    --sysconfdir=/etc \
    --with-dri-driverdir=/usr/libx32/xorg/modules/dri \
    --with-gallium-drivers=r300,r600,radeonsi,nouveau,swrast \
    --with-dri-drivers=i915,i965,r200,radeon,nouveau,swrast \
    --enable-gallium-llvm \
    --disable-gallium-egl --enable-shared-glapi \
    --enable-shared-glapi \
    --enable-glx-tls \
    --enable-dri \
    --enable-gles1 \
    --enable-gles2 \
    --disable-egl \
    --enable-texture-float \
    --enable-osmesa \
    --libdir=/usr/libx32 "
    #--enable-32-bit \
    # --enable-gbm disabled because it needs udev

  if [ "${_git}" = "true" ]; then
    ./autogen.sh \
      $COMMONOPTS
  else
    autoreconf -vfi
    ./configure \
      $COMMONOPTS
  fi

  make
}

package_libx32-libglapi() {
  depends=('libx32-glibc' 'libglapi')
  pkgdesc="free implementation of the GL API -- shared library. The Mesa GL API module is responsible for dispatching all the gl* functions (x32 ABI)"

  cd ${srcdir}/?esa-*   

  make -C src/mapi/shared-glapi DESTDIR="${pkgdir}" install

  install -m755 -d "${pkgdir}/usr/share/licenses/libglapi"
  ln -s libglapi "${pkgdir}/usr/share/licenses/libglapi/libx32-libglapi"
}

package_libx32-libgl() {
  depends=('libx32-libdrm>=2.4.39' 'libx32-libxxf86vm>=1.1.2' 'libx32-libxdamage>=1.1.3' 'libx32-expat>=2.1.0' 'libx32-libglapi'
           'libgl')
  pkgdesc="Mesa 3-D graphics library and DRI software rasterizer (x32 ABI)"
  # currently disabled so force the remove
  conflicts=('libx32-libgbm')
  replace=('libx32-libgbm')

  cd ${srcdir}/?esa-*

  # fix linking because of splitted package
  make -C src/mapi/shared-glapi DESTDIR="${pkgdir}" install

  # libGL & libdricore
  make -C src/glx DESTDIR="${pkgdir}" install
  make -C src/mesa/libdricore DESTDIR="${pkgdir}" install
  
  # fix linking because of splitted package - cleanup
  make -C src/mapi/shared-glapi DESTDIR="${pkgdir}" uninstall
  
  # --with-gallium-drivers=swrast
  make -C src/gallium/targets/dri-swrast DESTDIR="${pkgdir}" install

  install -m755 -d "${pkgdir}/usr/share/licenses/libgl"
  ln -s libgl "${pkgdir}/usr/share/licenses/libgl/libx32-libgl"
}

package_libx32-mesa() {
  # check also gl.pc
  depends=('libx32-libgl' 'libx32-libx11>=1.5.0' 'libx32-libxext>=1.3.1' 'libx32-libxdamage' 'libx32-libxfixes' 'libx32-libxcb'
           'libx32-libxxf86vm' 'mesa')
  pkgdesc="Mesa 3-D graphics libraries and include files (x32 ABI)"

  cd ${srcdir}/?esa-*

  # .pc files
  make -C src/mesa DESTDIR="${pkgdir}" install-pkgconfigDATA
  make -C src/mesa/drivers/dri DESTDIR="${pkgdir}" install-pkgconfigDATA

  install -m755 -d "${pkgdir}/usr/share/licenses/mesa"
  ln -s mesa "$pkgdir/usr/share/licenses/mesa/libx32-mesa"
}

package_libx32-osmesa() {
  depends=('libx32-libglapi' 'libx32-gcc-libs' 'osmesa')
  optdepends=('opengl-man-pages: for the OpenGL API man pages')
  pkgdesc="Mesa 3D off-screen rendering library (x32 ABI)"

  # fix linking because of splitted package
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" install

  make -C ${srcdir}/?esa-*/src/mesa/drivers/osmesa DESTDIR="${pkgdir}" install
  
  # fix linking because of splitted package - cleanup
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" uninstall
}

# package_lib32-libgbm() {
#   depends=('lib32-libglapi' 'lib32-libdrm' 'libgbm')
#   pkgdesc="Mesa gbm library (32-bit)"
# 
#   cd ${srcdir}/?esa-*
# 
#   # fix linking because of splitted package
#   make -C src/mapi/shared-glapi DESTDIR="${pkgdir}" install
# 
#   make -C src/gbm DESTDIR="${pkgdir}" install
# 
#   # fix linking because of splitted package - cleanup
#   make -C src/mapi/shared-glapi DESTDIR="${pkgdir}" uninstall
# 
#   install -m755 -d "${pkgdir}/usr/share/licenses/libgbm"
#   ln -s libgbm "$pkgdir/usr/share/licenses/libgbm/lib32-libgbm"
# }

package_libx32-libgles() {
  depends=('libx32-libglapi' 'libx32-libdrm' 'libgles')
  pkgdesc="Mesa GLES libraries (x32 ABI)"

  cd ${srcdir}/?esa-*

  # fix linking because of splitted package
  make -C src/mapi/shared-glapi DESTDIR="${pkgdir}" install

  # --enable-gles1 --enable-gles2
  make -C src/mapi/es1api DESTDIR="${pkgdir}" install
  make -C src/mapi/es2api DESTDIR="${pkgdir}" install

  # fix linking because of splitted package - cleanup
  make -C src/mapi/shared-glapi DESTDIR="${pkgdir}" uninstall
  
  rm -r "${pkgdir}"/usr/include
  
  install -m755 -d "${pkgdir}/usr/share/licenses/libgles"
  ln -s libgles "$pkgdir/usr/share/licenses/libgles/libx32-libgles"
}

package_libx32-ati-dri() {
  depends=("libx32-libgl=${pkgver}" 'ati-dri')
  pkgdesc="Mesa DRI radeon/r200 + Gallium3D for r300 and later chipsets drivers for AMD/ATI Radeon (x32 ABI)"
  conflicts=('xf86-video-ati<6.9.0-6')

  cd ${srcdir}/?esa-*
  
  # fix linking because of splitted package
  make -C src/mesa/libdricore DESTDIR="${pkgdir}" install

  # classic mesa drivers for radeon,r200
  make -C src/mesa/drivers/dri/radeon DESTDIR="${pkgdir}" install
  make -C src/mesa/drivers/dri/r200 DESTDIR="${pkgdir}" install
  # gallium3D driver for r300,r600
  make -C src/gallium/targets/dri-r300 DESTDIR="${pkgdir}" install
  make -C src/gallium/targets/dri-r600 DESTDIR="${pkgdir}" install
  make -C src/gallium/targets/dri-radeonsi DESTDIR="${pkgdir}" install
  
  # fix linking because of splitted package - cleanup
  make -C src/mesa/libdricore DESTDIR="${pkgdir}" uninstall
  
  install -m755 -d "${pkgdir}/usr/share/licenses/ati-dri"
  ln -s ati-dri "$pkgdir/usr/share/licenses/ati-dri/libx32-ati-dri"
}

package_libx32-intel-dri() {
  depends=("libx32-libgl=${pkgver}" 'intel-dri')
  pkgdesc="Mesa DRI drivers for Intel (x32 ABI)"

  cd ${srcdir}/?esa-*
  
  # fix linking because of splitted package
  make -C src/mesa/libdricore DESTDIR="${pkgdir}" install

  make -C src/mesa/drivers/dri/i915 DESTDIR="${pkgdir}" install
  make -C src/mesa/drivers/dri/i965 DESTDIR="${pkgdir}" install
  
  # fix linking because of splitted package - cleanup
  make -C src/mesa/libdricore DESTDIR="${pkgdir}" uninstall
  
  install -m755 -d "${pkgdir}/usr/share/licenses/intel-dri"
  ln -s intel-dri "$pkgdir/usr/share/licenses/intel-dri/libx32-intel-dri"
}

package_libx32-nouveau-dri() {
  depends=("libx32-libgl=${pkgver}" 'nouveau-dri')
  pkgdesc="Mesa classic DRI + Gallium3D drivers for Nouveau (x32 ABI)"

  cd ${srcdir}/?esa-*
  
  # fix linking because of splitted package
  make -C src/mesa/libdricore DESTDIR="${pkgdir}" install

  # classic mesa driver for nv10 , nv20 nouveau_vieux_dri.so
  make -C src/mesa/drivers/dri/nouveau DESTDIR="${pkgdir}" install
  # gallium3D driver for nv30 - nv40 - nv50 nouveau_dri.so
  make -C src/gallium/targets/dri-nouveau DESTDIR="${pkgdir}" install
  
  # fix linking because of splitted package - cleanup
  make -C src/mesa/libdricore DESTDIR="${pkgdir}" uninstall
  
  install -m755 -d "${pkgdir}/usr/share/licenses/nouveau-dri"
  ln -s nouveau-dri "$pkgdir/usr/share/licenses/nouveau-dri/libx32-nouveau-dri"
}

