# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://git.archlinux.org/svntogit/packages.git/log/trunk?h=packages/bluez
# 						Maintainer: Tom Gundersen <teg@jklm.no>
# 						Contributor: Andrea Scarpino <andrea@archlinux.org>
# 						Contributor: Geoffroy Carrier <geoffroy@archlinux.org>

pkgbase=bluez
pkgname=('bluez' 'bluez-utils' 'bluez-libs' 'bluez-cups' 'bluez-hid2hci' 'bluez-plugins')
pkgver=5.49
pkgrel=5
url="http://www.bluez.org/"
arch=(x86_64)
license=('GPL2')
makedepends=('dbus' 'libical' 'alsa-lib' 'json-c')
source=(https://www.kernel.org/pub/linux/bluetooth/${pkgname}-${pkgver}.tar.xz
        bluetooth.modprobe
        refresh_adv_manager_for_non-LE_devices.diff
        gatt_fix_crash.diff)
# see https://www.kernel.org/pub/linux/bluetooth/sha256sums.asc
sha256sums=('33301d7a514c73d535ee1f91c2aed1af1f2e53efe11d3ac06bcf0d7abed2ce95'
            '46c021be659c9a1c4e55afd04df0c059af1f3d98a96338236412e449bf7477b4'
            '5baa5b82f3b120f24c61b2e87870a9d8db44b15df9fe2557dad745894df2d501'
            'd66560f321ade2ecf6478850d1ddc1952e439c6fdd14ae885e49355bc9aa1e4c')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

prepare() {
  cd ${pkgname}-${pkgver}
  patch -Np1 -i ../refresh_adv_manager_for_non-LE_devices.diff
  patch -Np1 -i ../gatt_fix_crash.diff
}
build() {
  cd ${pkgname}-${pkgver}
  ./configure \
          --prefix=/usr \
          --mandir=/usr/share/man \
          --sysconfdir=/etc \
          --localstatedir=/var \
          --libexecdir=/usr/lib \
          --enable-sixaxis \
          --enable-mesh \
          --enable-experimental \
          --disable-systemd \
          --enable-library # this is deprecated
  make
}

check() {
  cd $pkgname-$pkgver
  make check || /bin/true # https://bugzilla.kernel.org/show_bug.cgi?id=196621
}


package_bluez() {
  pkgdesc="Daemons for the bluetooth protocol stack"
  depends=('libical' 'dbus' 'glib2' 'alsa-lib')
  backup=('etc/dbus-1/system.d/bluetooth.conf'
          'etc/bluetooth/main.conf')
  conflicts=('obexd-client' 'obexd-server')

  cd ${pkgbase}-${pkgver}
  make DESTDIR=${pkgdir} \
       install-libexecPROGRAMS \
       install-dbussessionbusDATA \
       install-dbussystembusDATA \
       install-dbusDATA \
       install-man8

  # ship upstream main config file
  install -dm755 ${pkgdir}/etc/bluetooth
  install -Dm644 ${srcdir}/${pkgbase}-${pkgver}/src/main.conf ${pkgdir}/etc/bluetooth/main.conf

  # add basic documention
  install -dm755 ${pkgdir}/usr/share/doc/${pkgbase}/dbus-apis
  cp -a doc/*.txt ${pkgdir}/usr/share/doc/${pkgbase}/dbus-apis/
  # fix module loading errors
  install -dm755 ${pkgdir}/usr/lib/modprobe.d
  install -Dm644 ${srcdir}/bluetooth.modprobe ${pkgdir}/usr/lib/modprobe.d/bluetooth-usb.conf	
  # load module at system start required by some functions
  # https://bugzilla.kernel.org/show_bug.cgi?id=196621
  install -dm755 $pkgdir/usr/lib/modules-load.d
  echo "crypto_user" > $pkgdir/usr/lib/modules-load.d/bluez.conf
}

package_bluez-utils() {
  pkgdesc="Development and debugging utilities for the bluetooth protocol stack"
  depends=('dbus' 'glib2')
  conflicts=('bluez-hcidump')
  provides=('bluez-hcidump')
  replaces=('bluez-hcidump' 'bluez<=4.101')

  cd ${pkgbase}-${pkgver}
  make DESTDIR=${pkgdir} \
       install-binPROGRAMS \
       install-man1

  # add missing tools FS#41132, FS#41687, FS#42716
  for files in `find tools/ -type f -perm -755`; do
    filename=$(basename $files)
    install -Dm755 ${srcdir}/${pkgbase}-${pkgver}/tools/$filename ${pkgdir}/usr/bin/$filename
  done
  
  # libbluetooth.so* are part of libLTLIBRARIES and binPROGRAMS targets
  #make DESTDIR=${pkgdir} uninstall-libLTLIBRARIES
  #rmdir ${pkgdir}/usr/lib
  rm -rf ${pkgdir}/usr/lib
  
  # move the hid2hci man page out
  mv ${pkgdir}/usr/share/man/man1/hid2hci.1 ${srcdir}/
}

package_bluez-libs() {
  pkgdesc="Deprecated libraries for the bluetooth protocol stack"
  depends=('glibc')
  license=('LGPL2.1')

  cd ${pkgbase}-${pkgver}
  make DESTDIR=${pkgdir} \
       install-includeHEADERS \
       install-libLTLIBRARIES \
       install-pkgconfigDATA
}

package_bluez-cups() {
  pkgdesc="CUPS printer backend for Bluetooth printers"
  depends=('cups')

  cd ${pkgbase}-${pkgver}
  make DESTDIR=${pkgdir} install-cupsPROGRAMS
}

package_bluez-hid2hci() {
  pkgdesc="Put HID proxying bluetooth HCI's into HCI mode"
  depends=()

  cd ${pkgbase}-${pkgver}
  make DESTDIR=${pkgdir} \
       install-udevPROGRAMS \
       install-rulesDATA
  
  install -dm755 ${pkgdir}/usr/share/man/man1
  mv ${srcdir}/hid2hci.1 ${pkgdir}/usr/share/man/man1/hid2hci.1
}

package_bluez-plugins() {
  pkgdesc="bluez plugins (PS3 Sixaxis controller)"
  depends=()

  cd ${pkgbase}-${pkgver}
  make DESTDIR=${pkgdir} \
       install-pluginLTLIBRARIES
}
