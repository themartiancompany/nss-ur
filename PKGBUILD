# Maintainer: Jan de Groot <jgc@archlinux.org>

pkgbase=nss
pkgname=(nss ca-certificates-mozilla)
pkgver=3.28.1
pkgrel=1
pkgdesc="Network Security Services"
url="https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS"
arch=(i686 x86_64)
license=('MPL' 'GPL')
_nsprver=4.12
depends=("nspr>=${_nsprver}" 'sqlite' 'zlib' 'sh' 'p11-kit')
makedepends=('perl' 'python2' 'xmlto' 'docbook-xsl')
options=('!strip' '!makeflags' 'staticlibs')
source=("https://ftp.mozilla.org/pub/mozilla.org/security/nss/releases/NSS_${pkgver//./_}_RTM/src/${pkgbase}-${pkgver}.tar.gz"
        certdata2pem.py bundle.sh nss.pc.in nss-config.in nss-config.xml)
sha256sums=('58cc0c05c0ed9523e6d820bea74f513538f48c87aac931876e3d3775de1a82ad'
            '2a2ff9131c21fa3b23ad7c7a2f069eabc783e56c6eb05419ac5f365f48dea0fc'
            '045f520403f715a4cc7f3607b4e2c9bcc88fee5bce58d462fddaa2fdb0e4c180'
            'f2208c4f70373ff9b60f53d733f8071d4e390c384b776dfc04bf26c306882faf'
            'e44ac5095b4d88f24ec7b2e6a9f1581560bd3ad41a3d198596d67ef22f67adb9'
            '98ace873c63e8e870286bce3ed53249aa2655cc1f53e7049061476e650ab06f1')

prepare() {
  mkdir certs

  echo -n "$(date +"%e %B %Y")" >date.xml
  echo -n "$pkgver" >version.xml

  cd nss-$pkgver

  # Respect LDFLAGS
  sed -e 's/\$(MKSHLIB) -o/\$(MKSHLIB) \$(LDFLAGS) -o/' \
      -i nss/coreconf/rules.mk

  ln -sr nss/lib/ckfw/builtins/certdata.txt ../certs/
  ln -sr nss/lib/ckfw/builtins/nssckbi.h ../certs/
}


build() {
  xmlto man nss-config.xml

  cd certs
  python2 ../certdata2pem.py

  cd ..
  sh bundle.sh

  cd nss-$pkgver/nss
  export BUILD_OPT=1
  export NSS_USE_SYSTEM_SQLITE=1
  export NSS_ALLOW_SSLKEYLOGFILE=1
  export NSS_ENABLE_ECC=1
  export NSPR_INCLUDE_DIR="`nspr-config --includedir`"
  export NSPR_LIB_DIR="`nspr-config --libdir`"
  export XCFLAGS="${CFLAGS}"

  [[ $CARCH == x86_64 ]] && export USE_64=1

  make -C coreconf
  make -C lib/dbm
  make
  make clean_docs build_docs
}

package_nss() {
  cd nss-$pkgver

  install -d "$pkgdir"/usr/{bin,include/nss,lib/pkgconfig,share/man/man1}

  NSS_VMAJOR=$(grep '#define.*NSS_VMAJOR' nss/lib/nss/nss.h | awk '{print $3}')
  NSS_VMINOR=$(grep '#define.*NSS_VMINOR' nss/lib/nss/nss.h | awk '{print $3}')
  NSS_VPATCH=$(grep '#define.*NSS_VPATCH' nss/lib/nss/nss.h | awk '{print $3}')

  sed ../nss.pc.in \
    -e "s,%libdir%,/usr/lib,g" \
    -e "s,%prefix%,/usr,g" \
    -e "s,%exec_prefix%,/usr/bin,g" \
    -e "s,%includedir%,/usr/include/nss,g" \
    -e "s,%NSPR_VERSION%,${_nsprver},g" \
    -e "s,%NSS_VERSION%,${pkgver},g" \
    > "$pkgdir/usr/lib/pkgconfig/nss.pc"
  ln -s nss.pc "$pkgdir/usr/lib/pkgconfig/mozilla-nss.pc"

  sed ../nss-config.in \
    -e "s,@libdir@,/usr/lib,g" \
    -e "s,@prefix@,/usr/bin,g" \
    -e "s,@exec_prefix@,/usr/bin,g" \
    -e "s,@includedir@,/usr/include/nss,g" \
    -e "s,@MOD_MAJOR_VERSION@,${NSS_VMAJOR},g" \
    -e "s,@MOD_MINOR_VERSION@,${NSS_VMINOR},g" \
    -e "s,@MOD_PATCH_VERSION@,${NSS_VPATCH},g" \
    > "$pkgdir/usr/bin/nss-config"
  chmod 755 "$pkgdir/usr/bin/nss-config"

  install -t "$pkgdir/usr/share/man/man1" -m644 nss/doc/nroff/*.1 ../nss-config.1

  cd dist
  install -t "$pkgdir/usr/include/nss" -m644 public/nss/*.h

  cd *.OBJ/bin
  install -t "$pkgdir/usr/bin" *util derdump pp shlibsign signtool signver ssltap vfychain vfyserv

  cd ../lib
  install -t "$pkgdir/usr/lib" *.so
  install -t "$pkgdir/usr/lib" -m644 *.chk libcrmf.a

  rm "$pkgdir/usr/lib/libnssckbi.so"
  ln -s libnssckbi-p11-kit.so "$pkgdir/usr/lib/libnssckbi.so"
}

package_ca-certificates-mozilla() {
  pkgdesc="Mozilla's set of trusted CA certificates"
  depends=(ca-certificates-utils)

  local _certdir="$pkgdir/usr/share/ca-certificates/trust-source"
  install -Dm644 ca-bundle.trust.crt "$_certdir/mozilla.trust.crt"
  install -Dm644 ca-bundle.neutral-trust.crt "$_certdir/mozilla.neutral-trust.crt"
  install -Dm644 ca-bundle.supplement.p11-kit "$_certdir/mozilla.supplement.p11-kit"
}
