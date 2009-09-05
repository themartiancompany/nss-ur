# Maintainer: Jan de Groot <jgc@archlinux.org>
pkgname=nss
pkgver=3.12.4
pkgrel=1
pkgdesc="Mozilla Network Security Services"
arch=(i686 x86_64)
url="http://www.mozilla.org/projects/security/pki/nss/"
license=('MPL' 'GPL')
_nsprver=4.8.0
depends=('nspr>=4.8' 'sqlite3>=3.6.17' 'zlib')
replaces=('nss-nspr')
source=(ftp://ftp.mozilla.org/pub/security/nss/releases/NSS_${pkgver//./_}_RTM/src/${pkgname}-${pkgver}.tar.gz
        nss-nolocalsql.patch
        nss-no-rpath.patch
        nss.pc.in
        nss-config.in)
md5sums=('1ee3ed9c1900079319bd1de51388d856'
         '1837781eed35bfb6f826cfb3efcd6409'
         'e5c97db0c884d5f4cfda21e562dc9bba'
         'c547b030c57fe1ed8b77c73bf52b3ded'
         '46bee81908f1e5b26d6a7a2e14c64d9f')

build() {
  cd "${srcdir}/${pkgname}-${pkgver}"
  patch -Np0 -i "${srcdir}/nss-nolocalsql.patch" || return 1
  patch -Np0 -i "${srcdir}/nss-no-rpath.patch" || return 1
  unset CFLAGS
  unset CXXFLAGS
  export BUILD_OPT=1
  export PKG_CONFIG_ALLOW_SYSTEM_LIBS=1
  export PKG_CONFIG_ALLOW_SYSTEM_CFLAGS=1
  export NSPR_INCLUDE_DIR=`pkg-config --cflags-only-I nspr | sed 's/-I//'`
  export NSPR_LIB_DIR=`pkg-config --libs-only-L nspr | sed 's/-L.//'`

  [ "$CARCH" = "x86_64" ] && export USE_64=1

  make -j 1 -C mozilla/security/coreconf || return 1
  make -j 1 -C mozilla/security/dbm || return 1
  make -j 1 -C mozilla/security/nss || return 1

  install -m755 -d "${pkgdir}/usr/lib/pkgconfig"
  install -m755 -d "${pkgdir}/usr/bin"
  install -m755 -d "${pkgdir}/usr/include/nss"

  NSS_VMAJOR=`grep "#define.*NSS_VMAJOR" mozilla/security/nss/lib/nss/nss.h | awk '{print $3}'`
  NSS_VMINOR=`grep "#define.*NSS_VMINOR" mozilla/security/nss/lib/nss/nss.h | awk '{print $3}'`
  NSS_VPATCH=`grep "#define.*NSS_VPATCH" mozilla/security/nss/lib/nss/nss.h | awk '{print $3}'`

  sed "${srcdir}/nss.pc.in" -e "s,%libdir%,/usr/lib,g" \
  				-e "s,%prefix%,/usr,g" \
				-e "s,%exec_prefix%,/usr/bin,g" \
				-e "s,%includedir%,/usr/include/nss,g" \
				-e "s,%NSPR_VERSION%,${pkgver},g" \
				-e "s,%NSS_VERSION%,${pkgver},g" > \
			"${pkgdir}/usr/lib/pkgconfig/nss.pc" || return 1
  ln -sf nss.pc "${pkgdir}/usr/lib/pkgconfig/mozilla-nss.pc" || return 1
  chmod 644 ${pkgdir}/usr/lib/pkgconfig/*.pc || return 1

  sed "${srcdir}/nss-config.in" -e "s,@libdir@,/usr/lib,g" \
  				    -e "s,@prefix@,/usr/bin,g" \
				    -e "s,@exec_prefix@,/usr/bin,g" \
				    -e "s,@includedir@,/usr/include/nss,g" \
				    -e "s,@MOD_MAJOR_VERSION@,${NSS_VMAJOR},g" \
				    -e "s,@MOD_MINOR_VERSION@,${NSS_VMINOR},g" \
				    -e "s,@MOD_PATCH_VERSION@,${NSS_VPATCH},g" \
			    > "${pkgdir}/usr/bin/nss-config" || return 1
  chmod 755 "${pkgdir}/usr/bin/nss-config" || return 1

  for file in libsoftokn3.so libfreebl3.so libnss3.so libnssutil3.so \
              libssl3.so libsmime3.so libnssckbi.so libnssdbm3.so
  do
    install -m755 mozilla/dist/*.OBJ/lib/${file} "${pkgdir}/usr/lib/" || return 1
  done
  for file in libcrmf.a libnssb.a libnssckfw.a; do
    install -m644 mozilla/dist/*.OBJ/lib/${file} "${pkgdir}/usr/lib/" || return 1
  done

  for file in certutil cmsutil crlutil modutil pk12util signtool signver ssltap; do
    install -m755 mozilla/dist/*.OBJ/bin/${file} "${pkgdir}/usr/bin/" || return 1
  done

  install -m644 mozilla/dist/public/nss/*.h "${pkgdir}/usr/include/nss/" || return 1
}
