# Maintainer: Matthias Erll <matthias@erll.de>

pkgname=pgadmin4-nw
pkgver=9.6
pkgrel=1
pkgdesc='Comprehensive design and management interface for PostgreSQL'
url='https://www.pgadmin.org/'
arch=('x86_64')
license=('custom')
depends=('postgresql-libs' 'hicolor-icon-theme' 'python'
         'libxcrypt' 'glibc' 'gcc-libs'
         'electron')
makedepends=('python-setuptools' 'python-virtualenv' 'yarn')
provides=('pgadmin4=9.6')
conflicts=('pgadmin4')
source=(https://ftp.postgresql.org/pub/pgadmin/pgadmin4/v${pkgver}/source/pgadmin4-${pkgver}.tar.gz{,.asc}
        pgAdmin4.desktop)
validpgpkeys=('E8697E2EEF76C02D3A6332778881B2A8210976F2') # Package Manager (Package Signing Key) <packages@pgadmin.org>
sha512sums=('1b681e27329b4979a00839bc83b8a0a1e8796c8a12f7cf8da27c44ed4cdea6ae176a34d7b6608f2b8f45ec396d1c65d5fdace94822edaf3eb5702dd022d5179b'
            'SKIP'
            '844868095bbfb40fd0abc641f92f609c71e1d2f87b6edae3b79735266731df0bd8d6bfd513d492ec6016fdd6eb73e4a153366e7e2456adc8173524eb6dab94e2')

prepare() {
  cd pgadmin4-${pkgver}

  # Create build environment
  if [ -d venv-build ]; then rm -R venv-build; fi
  python3 -m venv venv-build
  venv-build/bin/pip install --upgrade pip
  venv-build/bin/pip install wheel sphinx==8.1.3 sphinxcontrib-youtube -r requirements.txt

  # Create runtime environment (separate from build for reducing package size)
  if [ -d venv ]; then rm -R venv; fi
  python3 -m venv venv
  venv/bin/pip install --upgrade pip
  venv/bin/pip install -r requirements.txt
}

build() {
  export PGADMIN_PYTHON_DIR=/usr

  cd pgadmin4-${pkgver}
  source venv-build/bin/activate

  # override doctree directory
  make docs SPHINXOPTS='-d /tmp/'
  export CFLAGS+=" ${CPPFLAGS}"
  export CXXFLAGS+=" ${CPPFLAGS}"
  make runtime
  yarn set version berry
  yarn set version 3
  cd runtime && yarn plugin import workspace-tools && yarn workspaces focus --all --production && cd ..
  make install-node
  make bundle
  # Replace path references to virtual environment in build path
  find venv/bin -type f -exec sed -E "s|$(pwd)|/usr/lib/pgadmin4|g" -i '{}' +
  sed -E "s|$(pwd)|/usr/lib/pgadmin4|g" -i venv/pyvenv.cfg
  # Modify relative path to venv
  sed -E "s#(\.\./){5}(venv/bin/python3|web/pgAdmin4\.py)#../../../\2#g" -i runtime/src/js/misc.js
  # Remove cached files that contain references to src directory
  find . -type d -name __pycache__ -exec rm -R '{}' +
  # Remove additional dev and build files
  rm -R web/node_modules
  rm -R web/regression
  find web/pgadmin -type d -name "tests" -exec rm -R '{}' +
  rm web/pgadmin/messages.pot
  find web/pgadmin/translations -type f -name "messages.po" -delete
}

package() {
  cd pgadmin4-${pkgver}

  install -Dm 755 -d "${pkgdir}/usr/lib/pgadmin4/runtime"
  cp -a runtime/{assets,node_modules,src,package.json} "${pkgdir}/usr/lib/pgadmin4/runtime"
  cp -a web venv "${pkgdir}/usr/lib/pgadmin4"
  install -Dm 755 -d docs/en_US/_build/html "${pkgdir}/usr/lib/pgadmin4/share/docs/en_US"
  cp -a pkg/linux/config_distro.py "${pkgdir}/usr/lib/pgadmin4/web"

  install -Dm 644 pkg/linux/pgadmin4-128x128.png "${pkgdir}/usr/share/icons/hicolor/128x128/apps/pgAdmin4.png"
  install -Dm 644 pkg/linux/pgadmin4-64x64.png "${pkgdir}/usr/share/icons/hicolor/64x64/apps/pgAdmin4.png"
  install -Dm 644 pkg/linux/pgadmin4-48x48.png "${pkgdir}/usr/share/icons/hicolor/48x48/apps/pgAdmin4.png"
  install -Dm 644 pkg/linux/pgadmin4-32x32.png "${pkgdir}/usr/share/icons/hicolor/32x32/apps/pgAdmin4.png"
  install -Dm 644 pkg/linux/pgadmin4-16x16.png "${pkgdir}/usr/share/icons/hicolor/16x16/apps/pgAdmin4.png"
  install -Dm 644 "${srcdir}/pgAdmin4.desktop" -t "${pkgdir}/usr/share/applications"

  install -D /dev/stdin "${pkgdir}/usr/bin/pgadmin4" <<END
#!/bin/sh
exec /usr/bin/electron /usr/lib/pgadmin4/runtime "\$@"
END

  install -Dm 644 LICENSE -t "${pkgdir}/usr/share/licenses/${pkgbasename}"
}
