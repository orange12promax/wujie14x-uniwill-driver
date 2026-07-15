pkgname=wujie14x-uniwill-dkms
pkgver=0.2.0
pkgrel=1
pkgdesc='DKMS Uniwill platform driver extensions for MECHREVO WUJIE14XA'
arch=('x86_64')
url='https://github.com/torvalds/linux/tree/v7.1/drivers/platform/x86/uniwill'
license=('GPL-2.0-or-later')
depends=('dkms')
provides=('wujie14x-uniwill')
# The three Uniwill files are vendored from Linux v7.1 (commit
# 8cd9520d35a6c38db6567e97dd93b1f11f185dc6), then patched locally for the
# exact WUJIE14XA DMI match and features that have been verified on real
# hardware. Currently enabled: platform_profile support, read-only hwmon
# telemetry (CPU temperature, main-fan RPM and duty), plus Linux 6.18/7.1
# API compatibility.
source=(
  'uniwill-acpi.c'
  'uniwill-wmi.c'
  'uniwill-wmi.h'
  'Kbuild'
  'dkms.conf'
)
sha256sums=(
  'af73b5e700b08142161a10fb6a53bc984b937bf88c906dc8453d582019c0a4aa'
  'e730a13e2c3b5bc741d6a626a4b5ad6d23b8d6834657bc7752952db3108641d9'
  '30425f636c23e938b8d193753d5e486456fe23ea3d8b08300322115267f75374'
  '4c9c3e5b15677adf00dd69d2369aed2367647f789168574314491af809ed9e53'
  'd4db3474da2ed65ad001008c947dfe4976cbf15d8dfba0cea6b23e370d77f7e2'
)

package() {
  local srcdir_name="wujie14x-uniwill-${pkgver}"

  install -d "${pkgdir}/usr/src/${srcdir_name}"
  install -m644 Kbuild uniwill-acpi.c uniwill-wmi.c uniwill-wmi.h \
    "${pkgdir}/usr/src/${srcdir_name}/"
  sed "s/@PACKAGE_VERSION@/${pkgver}/" dkms.conf > \
    "${pkgdir}/usr/src/${srcdir_name}/dkms.conf"
}
