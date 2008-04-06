# $Id: PKGBUILD,v 1.13 2008/03/12 21:53:40 jgc Exp $
# Maintainer: Alexander Baldeck <alexander@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinux.org>
pkgname=nss
pkgver=3.11.9
pkgrel=1
pkgdesc="Mozilla Network Security Services"
arch=(i686 x86_64)
url="http://www.mozilla.org/projects/security/pki/nss/"
license=('MPL' 'GPL')
_nsprver=4.7
depends=("nspr>=${_nsprver}")
replaces=('nss-nspr')
source=(ftp://ftp.mozilla.org/pub/mozilla.org/security/nss/releases/NSS_3_11_9_RTM/src/${pkgname}-${pkgver}.tar.gz
	nss.pc.in
	nss-config.in)

build() {
  cd ${startdir}/src/${pkgname}-${pkgver}
  export BUILD_OPT=1
  export XCFLAGS=${CFLAGS}
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
				-e "s,%NSS_VERSION%,${pkgver},g" > \
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

  for file in libnss3.so libssl3.so libsmime3.so \
	      libsoftokn3.so libnssckbi.so \
	      libfreebl3.so; do
    install -m755 mozilla/dist/*.OBJ/lib/${file} ${startdir}/pkg/usr/lib/ || return 1
  done
  

  for file in libcrmf.a libnssb.a libnssckfw.a libfreebl3.chk libsoftokn3.chk; do
    install -m644 mozilla/dist/*.OBJ/lib/${file} ${startdir}/pkg/usr/lib/ || return 1
  done

  for file in certutil modutil pk12util signtool ssltap; do
    install -m755 mozilla/dist/*.OBJ/bin/${file} ${startdir}/pkg/usr/bin/ || return 1
  done

  install -m644 mozilla/dist/public/nss/*.h ${startdir}/pkg/usr/include/nss/ || return 1
}
