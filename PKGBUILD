# Maintainer: Jan de Groot <jgc@archlinux.org>

pkgbase=nss
pkgname=(nss ca-certificates-mozilla)
pkgver=3.31
pkgrel=2
pkgdesc="Network Security Services"
url="https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS"
arch=(i686 x86_64)
license=(MPL GPL)
_nsprver=4.15
depends=("nspr>=${_nsprver}" sqlite zlib sh p11-kit)
makedepends=(perl python2 xmlto docbook-xsl gyp)
options=(!strip !makeflags staticlibs)
source=("https://ftp.mozilla.org/pub/security/nss/releases/NSS_${pkgver//./_}_RTM/src/${pkgbase}-${pkgver}.tar.gz"
        certdata2pem.py bundle.sh nss.pc.in nss-config.in nss-config.xml)
sha256sums=('e90561256a3271486162c1fbe8d614d118c333d36a4455be2af8688bd420a65d'
            '512b12a2f13129be62c008b4df0153f527dd7d71c2c5183de99dfa2a1c49dd8a'
            '3bfadf722da6773bdabdd25bdf78158648043d1b7e57615574f189a88ca865dd'
            'f2208c4f70373ff9b60f53d733f8071d4e390c384b776dfc04bf26c306882faf'
            'e44ac5095b4d88f24ec7b2e6a9f1581560bd3ad41a3d198596d67ef22f67adb9'
            '98ace873c63e8e870286bce3ed53249aa2655cc1f53e7049061476e650ab06f1')

prepare() {
  mkdir certs path

  ln -s /usr/bin/python2 path/python

  echo -n "$(date +"%e %B %Y")" >date.xml
  echo -n "$pkgver" >version.xml
  xmlto man nss-config.xml

  cd nss-$pkgver

  ln -sr nss/lib/ckfw/builtins/certdata.txt ../certs/
  ln -sr nss/lib/ckfw/builtins/nssckbi.h ../certs/
}


build() {
  cd certs
  python2 ../certdata2pem.py

  cd ..
  sh bundle.sh

  cd nss-$pkgver/nss
  PATH="$srcdir/path:$PATH" ./build.sh --opt --system-sqlite --system-nspr -v
}

package_nss() {
  cd nss-$pkgver

  { read _vmajor; read _vminor; read _vpatch; } \
    < <(awk '/#define.*NSS_V(MAJOR|MINOR|PATCH)/ {print $3}' nss/lib/nss/nss.h)

  sed ../nss.pc.in \
    -e "s,%libdir%,/usr/lib,g" \
    -e "s,%prefix%,/usr,g" \
    -e "s,%exec_prefix%,/usr/bin,g" \
    -e "s,%includedir%,/usr/include/nss,g" \
    -e "s,%NSPR_VERSION%,${_nsprver},g" \
    -e "s,%NSS_VERSION%,${pkgver},g" |
    install -Dm644 /dev/stdin "$pkgdir/usr/lib/pkgconfig/nss.pc"
  ln -s nss.pc "$pkgdir/usr/lib/pkgconfig/mozilla-nss.pc"

  sed ../nss-config.in \
    -e "s,@libdir@,/usr/lib,g" \
    -e "s,@prefix@,/usr/bin,g" \
    -e "s,@exec_prefix@,/usr/bin,g" \
    -e "s,@includedir@,/usr/include/nss,g" \
    -e "s,@MOD_MAJOR_VERSION@,${_vmajor},g" \
    -e "s,@MOD_MINOR_VERSION@,${_vminor},g" \
    -e "s,@MOD_PATCH_VERSION@,${_vpatch},g" |
    install -D /dev/stdin "$pkgdir/usr/bin/nss-config"

  install -Dt "$pkgdir/usr/share/man/man1" -m644 nss/doc/nroff/*.1 ../nss-config.1

  cd dist
  install -Dt "$pkgdir/usr/include/nss" -m644 public/nss/*.h

  cd Release/bin
  install -Dt "$pkgdir/usr/bin" *util derdump pp shlibsign signtool signver ssltap vfychain vfyserv

  cd ../lib
  install -Dt "$pkgdir/usr/lib" *.so
  install -Dt "$pkgdir/usr/lib" -m644 *.chk

  ln -sf libnssckbi-p11-kit.so "$pkgdir/usr/lib/libnssckbi.so"
}

package_ca-certificates-mozilla() {
  pkgdesc="Mozilla's set of trusted CA certificates"
  depends=(ca-certificates-utils)

  install -Dm644 ca-bundle.trust.p11-kit \
    "$pkgdir/usr/share/ca-certificates/trust-source/mozilla.trust.p11-kit"
}
