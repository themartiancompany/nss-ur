# Maintainer: Jan de Groot <jgc@archlinux.org>

pkgname=nss
pkgver=3.12.11
pkgrel=2
pkgdesc="Mozilla Network Security Services"
arch=(i686 x86_64)
url="http://www.mozilla.org/projects/security/pki/nss/"
license=('MPL' 'GPL')
_nsprver=4.8.9
depends=("nspr>=${_nsprver}" 'sqlite3' 'zlib' 'sh')
makedepends=('perl')
replaces=('nss-nspr')
options=('!strip')
source=(ftp://ftp.mozilla.org/pub/security/nss/releases/NSS_${pkgver//./_}_RTM/src/${pkgname}-${pkgver}.tar.gz
        nss-no-rpath.patch
        nss.pc.in
        nss-config.in
        add_spi+cacert_ca_certs.patch
        ssl-renegotiate-transitional.patch)
md5sums=('ca0ca058380be200cf247ea2496b5352'
         'e5c97db0c884d5f4cfda21e562dc9bba'
         'c547b030c57fe1ed8b77c73bf52b3ded'
         '46bee81908f1e5b26d6a7a2e14c64d9f'
         '7f39c19b1dfd62d7db7d8bf19f156fed'
         'd83c7b61abb7e9f8f7bcd157183d1ade')

build() {
  cd "${srcdir}/${pkgname}-${pkgver}"
  # Adds the SPI Inc. and CAcert.org CA certificates - patch from Debian
  patch -Np1 -i "${srcdir}/add_spi+cacert_ca_certs.patch"
  # Adds transitional SSL renegotiate support - patch from Debian
  patch -Np1 -i "${srcdir}/ssl-renegotiate-transitional.patch"
  # Removes rpath
  patch -Np0 -i "${srcdir}/nss-no-rpath.patch"

  # Respect LDFLAGS
  sed -e 's/\$(MKSHLIB) -o/\$(MKSHLIB) \$(LDFLAGS) -o/g' \
      -i mozilla/security/coreconf/rules.mk

  cd mozilla/security/nss/lib/ckfw/builtins
  make generate

  cd "${srcdir}/${pkgname}-${pkgver}"
  export BUILD_OPT=1
  export PKG_CONFIG_ALLOW_SYSTEM_LIBS=1
  export PKG_CONFIG_ALLOW_SYSTEM_CFLAGS=1
  export NSS_USE_SYSTEM_SQLITE=1
  export NSPR_INCLUDE_DIR=`pkg-config --cflags-only-I nspr | sed 's/-I//'`
  export NSPR_LIB_DIR=`pkg-config --libs-only-L nspr | sed 's/-L.//'`
  export XCFLAGS="${CFLAGS}"

  [ "$CARCH" = "x86_64" ] && export USE_64=1

  make -j 1 -C mozilla/security/coreconf
  make -j 1 -C mozilla/security/dbm
  make -j 1 -C mozilla/security/nss
}

package() {
  cd "${srcdir}/${pkgname}-${pkgver}"
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
			"${pkgdir}/usr/lib/pkgconfig/nss.pc"
  ln -sf nss.pc "${pkgdir}/usr/lib/pkgconfig/mozilla-nss.pc"
  chmod 644 ${pkgdir}/usr/lib/pkgconfig/*.pc

  sed "${srcdir}/nss-config.in" -e "s,@libdir@,/usr/lib,g" \
  				    -e "s,@prefix@,/usr/bin,g" \
				    -e "s,@exec_prefix@,/usr/bin,g" \
				    -e "s,@includedir@,/usr/include/nss,g" \
				    -e "s,@MOD_MAJOR_VERSION@,${NSS_VMAJOR},g" \
				    -e "s,@MOD_MINOR_VERSION@,${NSS_VMINOR},g" \
				    -e "s,@MOD_PATCH_VERSION@,${NSS_VPATCH},g" \
			    > "${pkgdir}/usr/bin/nss-config"
  chmod 755 "${pkgdir}/usr/bin/nss-config"

  for file in libsoftokn3.so libfreebl3.so libnss3.so libnssutil3.so \
              libssl3.so libsmime3.so libnssckbi.so libnssdbm3.so
  do
    install -m755 mozilla/dist/*.OBJ/lib/${file} "${pkgdir}/usr/lib/"
  done

  install -m644 mozilla/dist/*.OBJ/lib/libcrmf.a "${pkgdir}/usr/lib/"
  install -m644 mozilla/dist/*.OBJ/lib/*.chk "${pkgdir}/usr/lib/"

  for file in certutil cmsutil crlutil modutil pk12util shlibsign signtool signver ssltap; do
    install -m755 mozilla/dist/*.OBJ/bin/${file} "${pkgdir}/usr/bin/"
  done

  install -m644 mozilla/dist/public/nss/*.h "${pkgdir}/usr/include/nss/"
}
