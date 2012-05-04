# Maintainer: Jan de Groot <jgc@archlinux.org>

pkgname=nss
pkgver=3.13.4
pkgrel=2
pkgdesc="Mozilla Network Security Services"
arch=(i686 x86_64)
url="http://www.mozilla.org/projects/security/pki/nss/"
license=('MPL' 'GPL')
_nsprver=4.9
depends=("nspr>=${_nsprver}" 'sqlite' 'zlib' 'sh')
makedepends=('perl')
options=('!strip')
source=(ftp://ftp.mozilla.org/pub/security/nss/releases/NSS_${pkgver//./_}_RTM/src/${pkgname}-${pkgver}.tar.gz
        nss-no-rpath.patch
        nss.pc.in
        nss-config.in
        add_spi+cacert_ca_certs.patch
        ssl-renegotiate-transitional.patch)
sha1sums=('c5a829c3bd56aa743457faf21469065f87c2db75'
          'c8fcdb153af9d39689243119adb475905a657284'
          'aa5b2c0aa38d3c1066d511336cf28d1333e3aebd'
          'cb744cc3e56b604e4754bc3c7d9f25bb9a0a136c'
          '3d89f29e321d7df7269b7ae6d219654543feaa6a'
          '8a964a744ba098711b80c0d279a2993524e8eb92')

build() {
  cd "${srcdir}/${pkgname}-${pkgver}/mozilla"
  # Adds the SPI Inc. and CAcert.org CA certificates - patch from Debian, modified to apply on certdata.txt only
  patch -Np2 -i "${srcdir}/add_spi+cacert_ca_certs.patch"
  # Adds transitional SSL renegotiate support - patch from Debian
  patch -Np2 -i "${srcdir}/ssl-renegotiate-transitional.patch"
  # Removes rpath
  patch -Np2 -i "${srcdir}/nss-no-rpath.patch"

  # Respect LDFLAGS
  sed -e 's/\$(MKSHLIB) -o/\$(MKSHLIB) \$(LDFLAGS) -o/g' \
      -i security/coreconf/rules.mk

  # Generate certdata.c from certdata.txt
  cd security/nss/lib/ckfw/builtins
  make generate

  cd "${srcdir}/${pkgname}-${pkgver}"
  export BUILD_OPT=1
  export PKG_CONFIG_ALLOW_SYSTEM_LIBS=1
  export PKG_CONFIG_ALLOW_SYSTEM_CFLAGS=1
  export NSS_USE_SYSTEM_SQLITE=1
  export NSS_ENABLE_ECC=1
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
