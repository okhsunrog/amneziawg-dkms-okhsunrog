# Maintainer: okhsunrog <d3g3v3@gmail.com>
# Based on amneziawg-dkms by Vladislav Minakov <v@minakov.pro>
# Fork carrying a compat patch for Linux >= 7.1 (ipv6_stub removal)

pkgname=amneziawg-dkms-okhsunrog
pkgdesc="AmneziaWG kernel module (DKMS), patched to build against Linux >= 7.1"
url="https://github.com/amnezia-vpn/amneziawg-linux-kernel-module"
arch=("x86_64")
pkgver=1.0.20260611
pkgrel=1
license=('GPLv2')
provides=("AMNEZIAWG-MODULE=${pkgver}" "amneziawg-dkms=${pkgver}")
conflicts=("amneziawg-dkms")
source=("amneziawg-dkms-$pkgver.tar.gz::https://github.com/amnezia-vpn/amneziawg-linux-kernel-module/archive/refs/tags/v${pkgver}.tar.gz"
        "linux-7.1-ipv6-stub.patch")
sha512sums=('3a99b7812b86087aa6f2c0af02a1c43aa6f540d025a1613d484930a99d3589c4ba2e6c2fb7f1b941357bf13855a56a220ff7c0688b22359f954b4dc689db0fdc'
            '58e16e765a229b1982b195ca8937f46acfb9d357f2a2e61c9aff87dfb0f31600b20e69b661d061290ba0b495b56e70b12885a4f336f7f6f4599b965fe41bb847')

prepare() {
cd "${srcdir}/amneziawg-linux-kernel-module-${pkgver}/src"
patch -Np1 -i "${srcdir}/linux-7.1-ipv6-stub.patch"
}

package() {
depends=("dkms" "wget")
cat > "${srcdir}/amneziawg-linux-kernel-module-${pkgver}/kernel-tree-scripts/prepare-sources.sh" <<'EOF'
#!/bin/bash -eux
kernel="${1%%[^0-9.]*}"
if [[ "$kernel" =~ .0$ ]]; then kernel="${kernel%.0}"; fi
kernel_major="${1%%[^0-9]*}"
wget "https://cdn.kernel.org/pub/linux/kernel/v${kernel_major}.x/linux-${kernel}.tar.xz" -O- | tar -xvJf - --wildcards linux-${kernel}/drivers/net/wireguard "linux-${kernel}/K*" linux-${kernel}/include/uapi/linux/
ln -sf linux-${kernel} kernel;
EOF
cat > "${srcdir}/amneziawg-linux-kernel-module-${pkgver}/kernel-tree-scripts/cleanup-sources.sh" <<'EOF'
#!/bin/bash
AWG_TEMP_DIR="$(cat /var/lib/amnezia/amneziawg/.tempdir 2>/dev/null)"
PREFIX=${AWG_TEMP_DIR:-/tmp}
WORKDIR="${PREFIX}/amneziawg"
[ -e kernel ] && rm -rf kernel
if [[ -d "${WORKDIR}" ]]; then
rm -rf "${WORKDIR}";
fi
EOF
cd ${srcdir}/amneziawg-linux-kernel-module-${pkgver}/src
sed -i 's/MODERN_KERNEL_SOURCES_NOT_FOUND_ERROR/KERNEL_SRC_ABSENT_ERR/g' Makefile
make DESTDIR=${pkgdir} dkms-install
}
