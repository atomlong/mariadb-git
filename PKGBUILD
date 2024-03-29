# Merged with official ABS mariadb PKGBUILD by João, 2021/07/22 (all respective contributors apply herein)
# Maintainer: João Figueiredo & chaotic-aur <islandc0der@chaotic.cx>
# Contributor: Bartłomiej Piotrowski <bpiotrowski@archlinux.org>
# Contributor: Christian Hesse <mail@eworm.de>

pkgbase=mariadb-git
pkgname=(mariadb-libs-git mariadb-clients-git mariadb-git mytop-git)
pkgdesc='Fast SQL database server, derived from MySQL'
pkgver=10.9.7_r197760.g717e3b3cfdb
pkgrel=1
arch=($CARCH)
license=(GPL)
url='https://mariadb.org/'
makedepends=(git boost bzip2 cmake jemalloc libaio libxcrypt libxml2 lz4 lzo openssl systemd zlib zstd curl krb5 cracklib)
conflicts=(${pkgbase%-git})
provides=(${pkgbase%-git})
source=("$pkgbase::git+https://github.com/MariaDB/server.git"
		0001-arch-specific.patch)
sha256sums=('SKIP'
            '3289efb3452d199aec872115f35da3f1d6fd4ce774615076690e9bc8afae1460')

pkgver() {
  cd $pkgbase
  _major_ver="$(grep -m1 'MYSQL_VERSION_MAJOR' VERSION | cut -d '=' -f2)"
  _minor_ver="$(grep -m1 'MYSQL_VERSION_MINOR' VERSION | cut -d '=' -f2)"
  _micro_ver="$(grep -m1 'MYSQL_VERSION_PATCH' VERSION | cut -d '=' -f2)"
  echo "${_major_ver}.${_minor_ver}.${_micro_ver}_r$(git rev-list --count HEAD).g$(git rev-parse --short HEAD)"
}

prepare() {
  cd $pkgbase

  # For some reason, it doesn't automatically checkout the default remote branch
  git checkout 10.9

  # Arch Linux specific patches:
  #  * enable PrivateTmp for a little bit more security
  #  * force preloading jemalloc for memory management
  #  * make systemd-tmpfiles create MYSQL_DATADIR
  patch -Np1 < ../0001-arch-specific.patch
}

build() {
  local _cmake_options=(
    # build options
    -DCOMPILATION_COMMENT="Arch Linux"
    -DCMAKE_BUILD_TYPE=RelWithDebInfo
    -Wno-dev

    # file paths
    # /etc
    -DINSTALL_SYSCONFDIR=/etc
    -DINSTALL_SYSCONF2DIR=/etc/my.cnf.d
    # /run
    -DINSTALL_UNIX_ADDRDIR=/run/mysqld/mysqld.sock
    # /usr
    -DCMAKE_INSTALL_PREFIX=/usr
    # /usr/bin /usr/include
    -DINSTALL_SCRIPTDIR=bin
    -DINSTALL_INCLUDEDIR=include/mysql
    # /usr/lib
    -DINSTALL_PLUGINDIR=lib/mysql/plugin
    -DINSTALL_SYSTEMD_UNITDIR=/usr/lib/systemd/system/
    -DINSTALL_SYSTEMD_SYSUSERSDIR=/usr/lib/sysusers.d/
    -DINSTALL_SYSTEMD_TMPFILESDIR=/usr/lib/tmpfiles.d/
    # /usr/share
    -DINSTALL_SHAREDIR=share
    -DINSTALL_SUPPORTFILESDIR=share/mysql
    -DINSTALL_MYSQLSHAREDIR=share/mysql
    -DINSTALL_DOCREADMEDIR=share/doc/mariadb
    -DINSTALL_DOCDIR=share/doc/mariadb
    -DINSTALL_MANDIR=share/man
    # /var
    -DMYSQL_DATADIR=/var/lib/mysql

    # default settings
    -DDEFAULT_CHARSET=utf8mb4
    -DDEFAULT_COLLATION=utf8mb4_unicode_ci

    # features
    -DENABLED_LOCAL_INFILE=ON
    -DPLUGIN_EXAMPLE=NO
    -DPLUGIN_FEDERATED=NO
    -DPLUGIN_FEEDBACK=NO
    -DWITH_EMBEDDED_SERVER=ON
    -DWITH_EXTRA_CHARSETS=complex
    -DWITH_JEMALLOC=ON
    -DWITH_LIBWRAP=OFF
    -DWITH_PCRE=bundled
    -DWITH_READLINE=ON
    -DWITH_SSL=system
    -DWITH_SYSTEMD=yes
    -DWITH_UNIT_TESTS=OFF
    -DWITH_ZLIB=system
  )

  cmake -B build -S $pkgbase "${_cmake_options[@]}"

  cmake --build build
}

## Takes *really* long, so disabled by default.
# check() {
#   cd build/mysql-test
#   ./mtr --parallel=5 --mem --force --max-test-fail=0
# }

package_mariadb-libs-git() {
  pkgdesc='MariaDB libraries'
  depends=(bzip2 libaio libxcrypt libcrypt.so lz4 lzo openssl xz zlib)
  optdepends=('krb5: for gssapi authentication')
  conflicts=(mariadb-libs libmysqlclient{,-git} libmariadbclient{,-git} mariadb-connector-c{,-git})
  provides=(mariadb-libs libmariadbclient{,-git} mariadb-connector-c{,-git} libmariadb.so libmariadbd.so)
  replaces=(libmariadbclient-git)
  options=('staticlibs')

  for dir in libmariadb libmysqld libservices include; do
    DESTDIR="$pkgdir" cmake --install "build/$dir"
  done

  ln -s build/mariadb_config "$pkgdir"/usr/bin/mariadb-config
  ln -s build/mariadb_config "$pkgdir"/usr/bin/mysql_config
  install -D -m0644 $pkgbase/man/mysql_config.1 "$pkgdir"/usr/share/man/man1/mysql_config.1

  install -D -m0644 build/support-files/mariadb.pc "$pkgdir"/usr/share/pkgconfig/mariadb.pc
  install -D -m0644 $pkgbase/support-files/mysql.m4 "$pkgdir"/usr/share/aclocal/mysql.m4

  cd "$pkgdir"

  # remove man pages
  rm -r usr/share/man
}

package_mariadb-clients-git() {
  pkgdesc='MariaDB client tools'
  depends=(mariadb-libs-git=$pkgver jemalloc)
  conflicts=(mysql-clients{,-git})
  provides=(mysql-clients=$pkgver)

  DESTDIR="$pkgdir" cmake --install build/client

  # install man pages
  for man in mysql mysql_plugin mysql_upgrade mysqladmin mysqlbinlog mysqlcheck mysqldump mysqlimport mysqlshow mysqlslap mysqltest; do
    install -D -m0644 $pkgbase/man/"$man.1" "$pkgdir"/usr/share/man/man1/"$man.1"
  done
}

package_mariadb-git() {
  pkgdesc='Fast SQL database server, derived from MySQL'
  backup=('etc/my.cnf'
          'etc/my.cnf.d/client.cnf'
          'etc/my.cnf.d/enable_encryption.preset'
          'etc/my.cnf.d/mysql-clients.cnf'
          'etc/my.cnf.d/server.cnf'
          'etc/my.cnf.d/s3.cnf'
          'etc/my.cnf.d/spider.cnf'
          'etc/security/user_map.conf')
  install=mariadb.install
  depends=(mariadb-clients-git=$pkgver systemd-libs libxml2 zstd)
  optdepends=('cracklib: for cracklib plugin'
              'curl: for ha_s3 plugin'
              'galera: for MariaDB cluster with Galera WSREP'
              'python-mysqlclient: for myrocks_hotbackup'
              'perl-dbd-mariadb: for mariadb-hotcopy, mariadb-convert-table-format and mariadb-setpermission')
  conflicts=(mysql{,-git})
  provides=(mysql=$pkgver)
  options=(emptydirs)

  DESTDIR="$pkgdir" cmake --install build

  cd "$pkgdir"

  # no SysV init, please!
  rm -r etc/logrotate.d
  rm usr/bin/rcmysql
  rm usr/share/mysql/{binary-configure,mysql{,d_multi}.server}

  # move to proper licenses directories
  install -d usr/share/licenses/mariadb
  mv usr/share/doc/mariadb/COPYING* usr/share/licenses/mariadb/

  # move it where one might look for it
  mv usr/share/{groonga{,-normalizer-mysql},doc/mariadb/}

  # move to pam directories
  install -d {etc,usr/lib}/security
  mv usr/share/user_map.conf etc/security/
  mv usr/share/pam_user_map.so usr/lib/security/

  # already installed to real systemd unit directory or useless
  rm -r usr/share/mysql/systemd/
  rm -r usr/lib/systemd/system/mariadb@bootstrap.service.d

  # provided by mariadb-libs
  rm usr/bin/{mariadb{_,-},mysql_}config
  rm -r usr/include/
  rm usr/share/man/man1/mysql_config.1
  rm -r usr/share/aclocal
  rm usr/lib/lib*
  rm -r usr/lib/pkgconfig
  rm usr/lib/mysql/plugin/{auth_gssapi_client,caching_sha2_password,client_ed25519,dialog,mysql_clear_password,sha256_password}.so

  # provided by mariadb-clients
  rm usr/bin/mysql{,_plugin,_upgrade,admin,binlog,check,dump,import,show,slap,test}
  rm usr/bin/mariadb{,-{admin,binlog,check,conv,dump,import,plugin,show,slap,test,upgrade}}
  rm usr/share/man/man1/mysql{,_plugin,_upgrade,admin,binlog,check,dump,import,show,slap,test}.1

  # provided by mytop
  rm usr/bin/mytop

  # not needed
  rm -r usr/{mysql-test,sql-bench}
  rm usr/share/man/man1/mysql-test-run.pl.1
}

package_mytop-git() {
  pkgdesc='Top clone for MariaDB'
  depends=(perl perl-dbd-mariadb perl-term-readkey)

  install -D -m0755 build/scripts/mytop "$pkgdir"/usr/bin/mytop
}
