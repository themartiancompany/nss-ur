# Maintainer: Alexander Baldeck <alexander@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinux.org>
pkgname=nss
pkgver=3.12rc4
_nssver=3.12.0.3
pkgrel=1
pkgdesc="Mozilla Network Security Services"
arch=(i686 x86_64)
url="http://www.mozilla.org/projects/security/pki/nss/"
license=('MPL' 'GPL')
_nsprver=4.7.1
depends=("nspr>=${_nsprver}" 'sqlite3>=3.5.9-2')
replaces=('nss-nspr')
source=(#ftp://ftp.mozilla.org/pub/mozilla.org/security/nss/releases/NSS_3_11_9_RTM/src/${pkgname}-${pkgver}.tar.gz
	ftp://ftp.archlinux.org/other/nss/nss-${pkgver}.tar.bz2
	nss-nolocalsql.patch
	nss.pc.in
	nss-config.in)
md5sums=('9a18090d4f43f3019246d0a1cd55925c'
         '1837781eed35bfb6f826cfb3efcd6409'
         'c547b030c57fe1ed8b77c73bf52b3ded'
         '46bee81908f1e5b26d6a7a2e14c64d9f')

build() {
  patch -Np0 -i ${startdir}/src/nss-nolocalsql.patch || return 1
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

  install -m755 -d ${startdir}/pkg/usr/lib/pkgconfig
  install -m755 -d ${startdir}/pkg/usr/bin
  install -m755 -d ${startdir}/pkg/usr/include/nss

  NSS_VMAJOR=`grep "#define.*NSS_VMAJOR" mozilla/security/nss/lib/nss/nss.h | awk '{print $3}'`
  NSS_VMINOR=`grep "#define.*NSS_VMINOR" mozilla/security/nss/lib/nss/nss.h | awk '{print $3}'`
  NSS_VPATCH=`grep "#define.*NSS_VPATCH" mozilla/security/nss/lib/nss/nss.h | awk '{print $3}'`

  sed ${startdir}/src/nss.pc.in -e "s,%libdir%,/usr/lib,g" \
  				-e "s,%prefix%,/usr,g" \
				-e "s,%exec_prefix%,/usr/bin,g" \
				-e "s,%includedir%,/usr/include/nss,g" \
				-e "s,%NSPR_VERSION%,${_nsprver},g" \
				-e "s,%NSS_VERSION%,${_nssver},g" > \
			${startdir}/pkg/usr/lib/pkgconfig/nss.pc || return 1
  ln -sf nss.pc ${startdir}/pkg/usr/lib/pkgconfig/mozilla-nss.pc || return 1
  chmod 644 ${startdir}/pkg/usr/lib/pkgconfig/*.pc || return 1

  sed ${startdir}/src/nss-config.in -e "s,@libdir@,/usr/lib,g" \
  				    -e "s,@prefix@,/usr/bin,g" \
				    -e "s,@exec_prefix@,/usr/bin,g" \
				    -e "s,@includedir@,/usr/include/nss,g" \
				    -e "s,@MOD_MAJOR_VERSION@,${NSS_VMAJOR},g" \
				    -e "s,@MOD_MINOR_VERSION@,${NSS_VMINOR},g" \
				    -e "s,@MOD_PATCH_VERSION@,${NSS_VPATCH},g" \
			    > ${startdir}/pkg/usr/bin/nss-config || return 1
  chmod 755 ${startdir}/pkg/usr/bin/nss-config || return 1

  for file in libsoftokn3.so libfreebl3.so libnss3.so libnssutil3.so \
              libssl3.so libsmime3.so libnssckbi.so libnssdbm3.so
  do
    install -m755 mozilla/dist/*.OBJ/lib/${file} ${startdir}/pkg/usr/lib/ || return 1
  done
  for file in libcrmf.a libnssb.a libnssckfw.a; do
    install -m644 mozilla/dist/*.OBJ/lib/${file} ${startdir}/pkg/usr/lib/ || return 1
  done

  for file in certutil cmsutil crlutil modutil pk12util signtool signver ssltap; do
    install -m755 mozilla/dist/*.OBJ/bin/${file} ${startdir}/pkg/usr/bin/ || return 1
  done

  install -m644 mozilla/dist/public/nss/*.h ${startdir}/pkg/usr/include/nss/ || return 1
}
