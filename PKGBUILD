# Maintainer: Matthias Erll <matthias@erll.de>

pkgname=pgadmin4-nw
pkgver=6.3
pkgrel=2
pkgdesc='Comprehensive design and management interface for PostgreSQL'
url='https://www.pgadmin.org/'
arch=('x86_64')
license=('custom')
depends=('postgresql-libs' 'hicolor-icon-theme' 'python'
         'libxcrypt' 'glibc' 'gcc-libs'
         'python-flask' 'python-flask-gravatar' 'python-flask-login'
         'python-flask-mail' 'python-flask-migrate' 'python-flask-sqlalchemy'
         'python-flask-wtf' 'python-flask-compress' 'python-flask-paranoid'
         'python-flask-babel' 'python-flask-security-too' 'python-flask-socketio'
         'python-wtforms' 'python-passlib' 'python-pytz' 'python-simplejson'
         'python-six' 'python-speaklater' 'python-sqlparse' 'python-psutil'
         'python-psycopg2' 'python-dateutil' 'python-sqlalchemy' 'python-bcrypt'
         'python-cryptography' 'python-sshtunnel' 'python-ldap3' 'python-gssapi'
         'python-eventlet' 'python-httpagentparser' 'python-user-agents'
         'python-authlib' 'python-requests' 'python-pyotp' 'python-qrcode'
         'python-pillow' 'nwjs-bin')
makedepends=('python-setuptools' 'python-sphinx' 'yarn')
provides=('pgadmin4=6.3')
conflicts=('pgadmin4')
source=(https://ftp.postgresql.org/pub/pgadmin/pgadmin4/v${pkgver}/source/pgadmin4-${pkgver}.tar.gz{,.asc}
        pgAdmin4.desktop)
validpgpkeys=('E8697E2EEF76C02D3A6332778881B2A8210976F2') # Package Manager (Package Signing Key) <packages@pgadmin.org>
sha512sums=('746f337dce7c97b35cf77a77713bb549a686943bff945026b9fb7e19bcffe2393ae5b1c91f91e21631317147d178ca1b95c587769fb5acf14371d6f961eb1f2d'
            'SKIP'
            'd061d074419b78ed96600329c622334310ca8fdef4b7c68d2594eb322ba814e21f4ce54daa8a27f3ce48a643c72feb342f7258eba52db6f915dff6a73bdba7da')

prepare() {
  cd pgadmin4-${pkgver}

  sed -E -i requirements.txt \
    -e '/Flask>?=/d' \
    -e '/Flask-Gravatar>?=/d' \
    -e '/Flask-Login>?=/d' \
    -e '/Flask-Mail>?=/d' \
    -e '/Flask-Migrate>?=/d' \
    -e '/Flask-SQLAlchemy>?=/d' \
    -e '/Flask-WTF>?=/d' \
    -e '/Flask-Compress>?=/d' \
    -e '/Flask-Paranoid>?=/d' \
    -e '/Flask-Babel>?=/d' \
    -e '/Flask-Security-Too>?=/d' \
    -e '/Flask-SocketIO>?=/d' \
    -e '/WTForms>?=/d' \
    -e '/passlib>?=/d' \
    -e '/pytz>?=/d' \
    -e '/simplejson>?=/d' \
    -e '/six>?=/d' \
    -e '/speaklater3>?=/d' \
    -e '/sqlparse>?=/d' \
    -e '/psutil>?=/d' \
    -e '/psycopg2>?=/d' \
    -e '/python-dateutil>?=/d' \
    -e '/SQLAlchemy>?=/d' \
    -e '/bcrypt>?=/d' \
    -e '/cryptography>?=/d' \
    -e '/sshtunnel>?=/d' \
    -e '/ldap3>?=/d' \
    -e '/gssapi>?=/d' \
    -e '/eventlet>?=/d' \
    -e '/httpagentparser>?=/d' \
    -e '/user-agents>?=/d' \
    -e '/pywinpty>?=/d' \
    -e '/Authlib>?=/d' \
    -e '/requests>?=/d' \
    -e '/pyotp>?=/d' \
    -e '/qrcode>?=/d' \
    -e '/Pillow>?=/d' \
    -e '/^#.*/d' \
    -e '/^$/d'
  if [[ -s requirements.txt ]]; then
    echo "ERROR: requirements.txt must be empty:"
    cat requirements.txt
    exit 1
  fi
}

build() {
  export LANG=en_US.UTF-8
  export LC_ALL=en_US.UTF-8
  export PGADMIN_PYTHON_DIR=/usr

  cd pgadmin4-${pkgver}
  # override doctree directory
  make docs SPHINXOPTS='-d /tmp/'
  export CFLAGS+=" ${CPPFLAGS}"
  export CXXFLAGS+=" ${CPPFLAGS}"
  make runtime
  cd runtime && yarn install --production=true && cd ..
  make install-node
  make bundle

  # Replace path references to virtual environment
  sed -E "s|../venv/bin/python3|/usr/bin/python3|g" -i runtime/src/js/misc.js
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

  mkdir -p "${pkgdir}/usr/lib/pgadmin4/runtime"
  cp -a runtime/{assets,node_modules,src,package.json} "${pkgdir}/usr/lib/pgadmin4/runtime"
  cp -a docs web "${pkgdir}/usr/lib/pgadmin4"

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