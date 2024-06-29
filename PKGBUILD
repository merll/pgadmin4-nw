# Maintainer: Matthias Erll <matthias@erll.de>

pkgname=pgadmin4-nw
pkgver=8.9
pkgrel=1
pkgdesc='Comprehensive design and management interface for PostgreSQL'
url='https://www.pgadmin.org/'
arch=('x86_64')
license=('custom')
depends=('postgresql-libs' 'hicolor-icon-theme' 'python'
         'libxcrypt' 'glibc' 'gcc-libs'
         'nwjs-bin')
makedepends=('python-setuptools' 'python-virtualenv' 'yarn')
provides=('pgadmin4=8.9')
conflicts=('pgadmin4')
source=(https://ftp.postgresql.org/pub/pgadmin/pgadmin4/v${pkgver}/source/pgadmin4-${pkgver}.tar.gz{,.asc}
        pgAdmin4.desktop)
validpgpkeys=('E8697E2EEF76C02D3A6332778881B2A8210976F2') # Package Manager (Package Signing Key) <packages@pgadmin.org>
sha512sums=('60ee85727916df3d694fcf45c09f22247b27556ca2c345be8353c9d0f2583f927917c8398b4e2f69e6f17f4a22f30895f7113a5fb44eafb59852fd894dde5c59'
            'SKIP'
            'd061d074419b78ed96600329c622334310ca8fdef4b7c68d2594eb322ba814e21f4ce54daa8a27f3ce48a643c72feb342f7258eba52db6f915dff6a73bdba7da')

prepare() {
  cd pgadmin4-${pkgver}

  # Create build environment
  if [ -d venv-build ]; then rm -R venv-build; fi
  python3 -m venv venv-build
  venv-build/bin/pip install --upgrade pip
  venv-build/bin/pip install wheel sphinx==6.1.3 sphinxcontrib-youtube -r requirements.txt

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
  yarn set version stable
  cd runtime && yarn workspaces focus --all --production && cd ..
  make install-node
  make bundle
  # Replace path references to virtual environment in build path
  find venv/bin -type f -exec sed -E "s|$(pwd)|/usr/lib|g" -i '{}' +
  sed -E "s|$(pwd)|/usr/lib|g" -i venv/pyvenv.cfg
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
  cp -a docs web venv "${pkgdir}/usr/lib/pgadmin4"

  install -Dm 644 pkg/linux/pgadmin4-128x128.png "${pkgdir}/usr/share/icons/hicolor/128x128/apps/pgAdmin4.png"
  install -Dm 644 pkg/linux/pgadmin4-64x64.png "${pkgdir}/usr/share/icons/hicolor/64x64/apps/pgAdmin4.png"
  install -Dm 644 pkg/linux/pgadmin4-48x48.png "${pkgdir}/usr/share/icons/hicolor/48x48/apps/pgAdmin4.png"
  install -Dm 644 pkg/linux/pgadmin4-32x32.png "${pkgdir}/usr/share/icons/hicolor/32x32/apps/pgAdmin4.png"
  install -Dm 644 pkg/linux/pgadmin4-16x16.png "${pkgdir}/usr/share/icons/hicolor/16x16/apps/pgAdmin4.png"
  install -Dm 644 "${srcdir}/pgAdmin4.desktop" -t "${pkgdir}/usr/share/applications"

  install -D /dev/stdin "${pkgdir}/usr/bin/pgadmin4" <<END
#!/bin/sh
exec /usr/bin/nw /usr/lib/pgadmin4/runtime/ "\$@"
END

  install -Dm 644 LICENSE -t "${pkgdir}/usr/share/licenses/${pkgbasename}"
}
