# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://git.archlinux.org/svntogit/packages.git/log/trunk?h=packages/bluez
# 						Maintainer: Tom Gundersen <teg@jklm.no>
# 						Contributor: Andrea Scarpino <andrea@archlinux.org>
# 						Contributor: Geoffroy Carrier <geoffroy@archlinux.org>

pkgbase=bluez
pkgname=('bluez' 'bluez-utils' 'bluez-libs' 'bluez-cups' 'bluez-hid2hci' 'bluez-plugins')
pkgver=5.47
pkgrel=2
url="http://www.bluez.org/"
arch=(x86_64)
license=('GPL2')
makedepends=('dbus' 'libical')
source=(https://www.kernel.org/pub/linux/bluetooth/${pkgname}-${pkgver}.tar.xz
        bluetooth.modprobe)
# see https://www.kernel.org/pub/linux/bluetooth/sha256sums.asc
sha256sums=('cf75bf7cd5d564f21cc4a2bd01d5c39ce425397335fd47d9bbe43af0a58342c8'
            '46c021be659c9a1c4e55afd04df0c059af1f3d98a96338236412e449bf7477b4')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

build() {
  cd ${pkgname}-${pkgver}
  ./configure \
          --prefix=/usr \
          --mandir=/usr/share/man \
          --sysconfdir=/etc \
          --localstatedir=/var \
          --libexecdir=/usr/lib \
          --enable-sixaxis \
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
  depends=('libical' 'dbus' 'glib2')
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
