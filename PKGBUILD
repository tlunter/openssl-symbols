# Maintainer: Todd Lunter <tlunter@gmail.com>

_pkg=openssl
pkgname=openssl-symbols
_ver=1.0.1g
pkgver=1.0.1.g
pkgrel=1
pkgdesc='The Open Source toolkit for Secure Sockets Layer and Transport Layer Security'
arch=('i686' 'x86_64')
url='https://www.openssl.org'
license=('custom:BSD')
depends=('zlib' 'perl')
optdepends=('ca-certificates')
provides=('openssl')
conflicts=('openssl')
options=('!makeflags')
backup=('etc/ssl/openssl.cnf')
source=("https://www.openssl.org/source/${_pkg}-${_ver}.tar.gz"
        "https://www.openssl.org/source/${_pkg}-${_ver}.tar.gz.asc"
        'no-rpath.patch'
        'ca-dir.patch'
        'openssl-1.0.1f-perl-5.18.patch'
        'versioned-symbols.patch')
md5sums=('de62b43dfcd858e66a74bee1c834e959'
         'SKIP'
         'dc78d3d06baffc16217519242ce92478'
         '3bf51be3a1bbd262be46dc619f92aa90'
         'ea2a61c8bd43788d81d98f1ac36c98ac'
         '8352550c89279b85ebf92e5c19f97a48')

prepare() {
	cd $srcdir/$_pkg-$_ver

	# remove rpath: http://bugs.archlinux.org/task/14367
	patch -p0 -i $srcdir/no-rpath.patch
	# set ca dir to /etc/ssl by default
	patch -p0 -i $srcdir/ca-dir.patch
    # add OPENSSL_1.0.0 symbols
	patch -p0 -i $srcdir/versioned-symbols.patch

	patch -p1 -i $srcdir/openssl-1.0.1f-perl-5.18.patch
}

build() {
	cd $srcdir/$_pkg-$_ver

	if [ "${CARCH}" == 'x86_64' ]; then
		openssltarget='linux-x86_64'
		optflags='enable-ec_nistp_64_gcc_128'
	elif [ "${CARCH}" == 'i686' ]; then
		openssltarget='linux-elf'
		optflags=''
	fi

	# mark stack as non-executable: http://bugs.archlinux.org/task/12434
	./Configure --prefix=/usr --openssldir=/etc/ssl --libdir=lib \
		shared zlib ${optflags} \
		"${openssltarget}" \
		"-Wa,--noexecstack ${CFLAGS} ${LDFLAGS}"

	make depend
	make
}

check() {
	cd $srcdir/$_pkg-$_ver
	# the test fails due to missing write permissions in /etc/ssl
	# revert this patch for make test
	patch -p0 -R -i $srcdir/ca-dir.patch
	make test
	patch -p0 -i $srcdir/ca-dir.patch
}

package() {
	cd $srcdir/$_pkg-$_ver
	make INSTALL_PREFIX=$pkgdir MANDIR=/usr/share/man MANSUFFIX=ssl install
	install -D -m644 LICENSE $pkgdir/usr/share/licenses/$_pkg/LICENSE
}
