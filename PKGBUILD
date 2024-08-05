# SPDX-License-Identifier: AGPL-3.0
#
# Maintainer: Truocolo <truocolo@aol.com>
# Maintainer: Pellegrino Prevete (tallero) <pellegrinoprevete@gmail.com>
# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinux.org>

_py="python"
_os="$( \
  uname \
    -o)"
_arch="$( \
  uname \
    -m)"
_pkg=nss
pkgbase="${_pkg}"
pkgname=(
  "${_pkg}"
  "ca-certificates-mozilla"
)
pkgver=3.103
pkgrel=1
pkgdesc="Network Security Services"
url="https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS"
arch=(
  x86_64
  arm
  armv7l
  i686
  aarch64
  pentium4
  mips
  powerpc
)
license=(
  MPL-2.0
)
depends=(
  'nspr>=4.35'
  'p11-kit>=0.23.19'
  glibc
  sqlite
  zlib
)
if [[ "${_os}" == "GNU/Linux" ]]; then
  depends+=(
    sh  # nss-config script
  )
elif [[ "${_os}" == "Android" ]]; then
  depends+=(
    bash  # nss-config script
  )
fi
makedepends=(
  gyp
  mercurial
  perl
  "${_py}"
)
_url="https://hg.mozilla.org/projects/${_pkg}"
source=(
  "hg+${_url}#tag=NSS_${pkgver//./_}_RTM"
  bundle.sh
  certdata2pem.py
)
b2sums=(
  'e0fac43db87bc2881af9f078fc33d135fc2f25d7068fec02113395cb22249f4244e82b6385a60abb8cd8a66c4f738131e61321ba6859f563559be8b88f4a0e72'
  '4be5dd836c844fdd7b63302a6994d62149082c3bc81eef70f373f416fed80a61a923960e4390d1c391b81ab01b409370d788818a30ffdd3a4ed467b670f990f6'
  '6bb59dcc9289916dcbf8fb6d73db0c0cd7582dc12a3aa4e8be19ec62c9ede65fdd9470a2d92ec5a114506b78d2d21b8ae0a1b45a17dc1f90f7d75434a93da510'
)
if [[ "${_os}" == "Android" ]]; then
  source+=(
    '0001-lib-freebl-stubs.c.patch'
  )
  b2sums+=(
    'cca769395a6969d4d67119138cdb08424510485ef91bae15f88e48502f9622fa863d63f60b46bafb747b40be4866b296d28928cfc616e6562e8dab6e7dd81255'
  )
fi

prepare() {
  mkdir \
    -p \
    certs
  ln \
    -srft \
    certs \
    "${_pkg}/lib/ckfw/builtins/"{certdata.txt,"${_pkg}ckbi.h"}
  patch \
    -Np1 \
    -i \
    '0001-lib-freebl-stubs.c.patch'
}

_get_usr() {
  local \
    _cc \
    _bin \
    _usr
  _cc="$( \
    command \
      -v \
      "cc")"
  _bin="$( \
    dirname \
      "${_cc}")"
  _usr="$( \
    dirname \
      "${_bin}")"
  echo \
    "${_usr}"
}

build() {
  local \
    _buildsh_options=() \
    _cppflags=() \
    _cflags=() \
    _ldflags=() \
    _cxxflags=()
  _buildsh_options=(
    --disable-tests
    --enable-libpkix
    --opt
    --system-nspr
    --system-sqlite
  )
  if [[ "${_arch}" == "x86_64" ]]; then
    _buildsh_options+=(
      --target x64
    )
  elif [[ "${_arch}" == "aarch64" ]]; then
    _buildsh_options+=(
      USE_64=1
    )
  fi
  if [[ "${_os}" == "Android" ]]; then
    _cppflags+=(
      -DANDROID
    )
    _cflags+=(
      "${CFLAGS}"
      -I"$(_get_usr)/include/nspr"
      -Wno-unused-const-variable # mm
      -Wno-tautological-unsigned-char-zero-compare # wow
      -Wno-int-conversion
    )
    _cxxflags+=(
      "${CXXFLAGS}"
      -I"$(_get_usr)/include/nspr"
    )
    _ldflags+=(
      "${LDFLAGS}"
      -llog
    )
  fi
  cd \
    certs
  ../certdata2pem.py
  cd \
    ..
  ./bundle.sh
  cd \
    "${_pkg}"
  CPPFLAGS="${_cppflags[*]}" \
  CFLAGS="${_cflags[*]}" \
  CXXFLAGS="${_cxxflags[*]}" \
  LDFLAGS="${_ldflags[*]}" \
  ./build.sh \
    "${_buildsh_options[@]}"
}

package_nss() {
  local \
    nsprver \
    libdir="/usr/lib" \
    includedir="/usr/include/${_pkg}" \
    includedir \
    vmajor \
    vminor \
    vpatch
  nsprver="$( \
    pkg-config \
      --modversion \
      nspr)"
  sed \
    "${_pkg}/pkg/pkg-config/${_pkg}.pc.in" \
    -e \
      "s,%prefix%,/usr,g" \
    -e \
      "s,%exec_prefix%,\${prefix},g" \
    -e \
      "s,%libdir%,${libdir},g" \
    -e \
      "s,%includedir%,${includedir},g" \
    -e \
      "s,%NSPR_VERSION%,${nsprver},g" \
    -e \
      "s,%NSS_VERSION%,${pkgver},g" |
    install \
      -Dm644 \
      /dev/stdin \
      "${pkgdir}${libdir}/pkgconfig/${_pkg}.pc"
  ln \
    -s \
    "${_pkg}.pc" \
    "${pkgdir}${libdir}/pkgconfig/mozilla-${_pkg}.pc"
  install \
    -Dt \
    "${pkgdir}${libdir}/" \
    dist/Release/lib/*.so || \

  { read vmajor; read vminor; read vpatch; } < \
    <(awk \
        '/#define.*NSS_V(MAJOR|MINOR|PATCH)/ {print $3}' \
        "${_pkg}/lib/${_pkg}/${_pkg}.h")
  sed \
    "${_pkg}/pkg/pkg-config/${_pkg}-config.in" \
    -e \
      "s,@prefix@,/usr,g" \
    -e \
      "s,@exec_prefix@,/usr,g" \
    -e \
      "s,@libdir@,${libdir},g" \
    -e \
      "s,@includedir@,${includedir},g" \
    -e \
      "s,@MOD_MAJOR_VERSION@,${vmajor},g" \
    -e \
      "s,@MOD_MINOR_VERSION@,${vminor},g" \
    -e \
      "s,@MOD_PATCH_VERSION@,${vpatch},g" |
    install \
      -D /dev/stdin \
      "${pkgdir}/usr/bin/${_pkg}-config"
  install \
    -Dt \
    "${pkgdir}/usr/bin" \
    dist/Release/bin/{*util,shlibsign,signtool,signver,ssltap}
  install \
    -Dt \
    "${pkgdir}${includedir}" \
    -m644 \
    "dist/public/${_pkg}/"*.h
  install \
    -Dt \
    "${pkgdir}/usr/share/man/man1" \
    -m644 \
    "${_pkg}/doc/nroff/"{*util,signtool,signver,ssltap}.1
  # Replace built-in trust with p11-kit connection
  ln \
    -s \
    pkcs11/p11-kit-trust.so \
    "${pkgdir}${libdir}/p11-kit-trust.so"
  ln \
    -sf \
    p11-kit-trust.so \
    "${pkgdir}${libdir}/lib${_pkg}ckbi.so"
}

package_ca-certificates-mozilla() {
  pkgdesc="Mozilla's set of trusted CA certificates"
  depends=(
    'ca-certificates-utils>=20181109-3'
  )
  install \
    -Dm644 \
    ca-bundle.trust.p11-kit \
    "${pkgdir}/usr/share/ca-certificates/trust-source/mozilla.trust.p11-kit"
}

# vim:set sw=2 sts=-1 et:
