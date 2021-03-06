pkgbase=php
pkgname=('php'
         'php-cgi'
         'php-apache'
         'php-fpm'
         'php-embed'
         'php-phpdbg'
         'php-pear'
         'php-enchant'
         'php-gd'
         'php-imap'
         'php-intl'
         'php-ldap'
         'php-mcrypt'
         'php-mssql'
         'php-odbc'
         'php-pgsql'
         'php-pspell'
         'php-snmp'
         'php-sqlite'
         'php-tidy'
         'php-xsl')
pkgver=5.6.28
pkgrel=1
arch=('i686' 'x86_64')
license=('PHP')
url='http://www.php.net'
makedepends=('apache' 'c-client' 'postgresql-libs' 'libldap' 'postfix'
             'sqlite' 'unixodbc' 'net-snmp' 'libzip' 'enchant' 'file' 'freetds'
             'libmcrypt' 'tidyhtml' 'aspell' 'libltdl' 'gd' 'icu'
             'curl' 'libxslt' 'openssl' 'db' 'gmp' 'systemd')
checkdepends=('procps-ng')
source=("http://www.php.net/distributions/${pkgbase}-${pkgver}.tar.bz2"
        # "http://www.php.net/distributions/${pkgbase}-${pkgver}.tar.bz2.asc"
        'php.ini.patch' 'apache.conf' 'php-fpm.conf.in.patch'
        'logrotate.d.php-fpm' 'php-fpm.service' 'php-fpm.tmpfiles')
md5sums=('56758fdb5ff156a36fb600950b7999aa'
         # 'SKIP'
         '39eff6cc99dae4ec3b52125e6229de7e'
         '0677a10d2e721472d6fccb470356b322'
         '16b5e2e4da59f15bea4c2db78a7bc8dc'
         '25bc67ad828e8147a817410b68d8016c'
         'cc2940f5312ba42e7aa1ddfab74b84c4'
         'c60343df74f8e1afb13b084d5c0e47ed')
validpgpkeys=('6E4F6AB321FDC07F2C332E3AC2BF0BC433CFC8B3'
              '0BD78B5F97500D450838F95DFE857D9A90D90EC1')

prepare() {
	cd ${srcdir}/${pkgbase}-${pkgver}

	patch -p0 -i ${srcdir}/php.ini.patch
	patch -p0 -i ${srcdir}/php-fpm.conf.in.patch
	# Just because our Apache 2.4 is configured with a threaded MPM by default does not mean we want to build a ZTS PHP.
	# Let's supress this behaviour and build a SAPI that works fine with the prefork MPM.
	sed '/APACHE_THREADED_MPM=/d' -i sapi/apache2handler/config.m4 -i configure
    # http://stackoverflow.com/questions/35524420/building-php5-6-18-on-arch-linux-windows-test-not-skipped-despite-php-os-check
    rm ./ext/standard/tests/streams/stream_socket_enable_crypto-win32.phpt
}

build() {
	local _phpconfig="--srcdir=../${pkgbase}-${pkgver} \
		--config-cache \
		--prefix=/usr \
		--sbindir=/usr/bin \
		--sysconfdir=/etc/php \
		--localstatedir=/var \
		--with-layout=GNU \
		--with-config-file-path=/etc/php \
		--with-config-file-scan-dir=/etc/php/conf.d \
		--disable-rpath \
		--mandir=/usr/share/man \
		--without-pear \
		"

	local _phpextensions="--enable-bcmath=shared \
		--enable-calendar=shared \
		--enable-dba=shared \
		--enable-exif=shared \
		--enable-ftp=shared \
		--enable-gd-native-ttf \
		--enable-intl=shared \
		--enable-mbstring \
		--enable-opcache \
		--enable-phar=shared \
		--enable-posix=shared \
		--enable-shmop=shared \
		--enable-soap=shared \
		--enable-sockets=shared \
		--enable-sysvmsg=shared \
		--enable-sysvsem=shared \
		--enable-sysvshm=shared \
		--enable-zip=shared \
		--with-bz2=shared \
		--with-curl=shared \
		--with-db4=/usr \
		--with-enchant=shared,/usr \
		--with-fpm-systemd \
		--with-freetype-dir=/usr \
		--with-xpm-dir=/usr \
		--with-gd=shared,/usr \
		--with-gdbm \
		--with-gettext=shared \
		--with-gmp=shared \
		--with-iconv=shared \
		--with-icu-dir=/usr \
		--with-imap-ssl \
		--with-imap=shared \
		--with-kerberos=/usr \
		--with-jpeg-dir=/usr \
		--with-vpx-dir=/usr \
		--with-ldap=shared \
		--with-ldap-sasl \
		--with-libzip \
		--with-mcrypt=shared \
		--with-mhash \
		--with-mssql=shared \
		--with-mysql-sock=/run/mysqld/mysqld.sock \
		--with-mysql=shared,mysqlnd \
		--with-mysqli=shared,mysqlnd \
		--with-openssl=shared \
		--with-pcre-regex=/usr \
		--with-pdo-mysql=shared,mysqlnd \
		--with-pdo-odbc=shared,unixODBC,/usr \
		--with-pdo-pgsql=shared \
		--with-pdo-sqlite=shared,/usr \
		--with-pgsql=shared \
		--with-png-dir=/usr \
		--with-pspell=shared \
		--with-snmp=shared \
		--with-sqlite3=shared,/usr \
		--with-tidy=shared \
		--with-unixODBC=shared,/usr \
		--with-xmlrpc=shared \
		--with-xsl=shared \
		--with-zlib \
		"

	EXTENSION_DIR=/usr/lib/php/modules
	export EXTENSION_DIR
	PEAR_INSTALLDIR=/usr/share/pear
	export PEAR_INSTALLDIR

	cd ${srcdir}/${pkgbase}-${pkgver}

	# php
	mkdir ${srcdir}/build-php
	cd ${srcdir}/build-php
	ln -s ../${pkgbase}-${pkgver}/configure
	./configure ${_phpconfig} \
		--disable-cgi \
		--with-readline \
		--enable-pcntl \
		${_phpextensions}
	make

	# cgi and fcgi
	# reuse the previous run; this will save us a lot of time
	cp -a ${srcdir}/build-php ${srcdir}/build-cgi
	cd ${srcdir}/build-cgi
	./configure ${_phpconfig} \
		--disable-cli \
		--enable-cgi \
		${_phpextensions}
	make

	# apache
	cp -a ${srcdir}/build-php ${srcdir}/build-apache
	cd ${srcdir}/build-apache
	./configure ${_phpconfig} \
		--disable-cli \
		--with-apxs2 \
		${_phpextensions}
	make

	# fpm
	cp -a ${srcdir}/build-php ${srcdir}/build-fpm
	cd ${srcdir}/build-fpm
	./configure ${_phpconfig} \
		--disable-cli \
		--enable-fpm \
		--with-fpm-user=http \
		--with-fpm-group=http \
		${_phpextensions}
	make

	# embed
	cp -a ${srcdir}/build-php ${srcdir}/build-embed
	cd ${srcdir}/build-embed
	./configure ${_phpconfig} \
		--disable-cli \
		--enable-embed=shared \
		${_phpextensions}
	make

	# phpdbg
	cp -a ${srcdir}/build-php ${srcdir}/build-phpdbg
	cd ${srcdir}/build-phpdbg
	./configure ${_phpconfig} \
		--disable-cli \
		--disable-cgi \
		--with-readline \
		--enable-phpdbg \
		${_phpextensions}
	make

	# pear
	cp -a ${srcdir}/build-php ${srcdir}/build-pear
	cd ${srcdir}/build-pear
	./configure ${_phpconfig} \
		--disable-cgi \
		--with-readline \
		--enable-pcntl \
		--with-pear \
		${_phpextensions}
	make
}

# check() {
# 	cd ${srcdir}/${pkgbase}-${pkgver}
# 
# 	# tests on i686 fail
# 	[[ $CARCH == 'i686' ]] && return
# 	# a couple of tests fail in btrfs-backed chroots
# 	[[ $(stat -f -c %T .) == btrfs ]] && return
# 
# 	export REPORT_EXIT_STATUS=1
# 	export NO_INTERACTION=1
# 	export SKIP_ONLINE_TESTS=1
# 	export SKIP_SLOW_TESTS=1
# 
# 	${srcdir}/build-php/sapi/cli/php -n \
# 		run-tests.php -n -P \
# 		{tests,Zend,ext/{spl,standard},sapi/cli}
# }

package_php() {
	pkgdesc='An HTML-embedded scripting language'
	depends=('pcre' 'libxml2' 'curl' 'libzip')
	backup=('etc/php/php.ini')

	cd ${srcdir}/build-php
	make -j1 INSTALL_ROOT=${pkgdir} install
	install -d -m755 ${pkgdir}/usr/share/pear
	# install php.ini
	install -D -m644 ${srcdir}/${pkgbase}-${pkgver}/php.ini-production ${pkgdir}/etc/php/php.ini
	install -d -m755 ${pkgdir}/etc/php/conf.d/

	# remove static modules
	rm -f ${pkgdir}/usr/lib/php/modules/*.a
	# remove modules provided by sub packages
	rm -f ${pkgdir}/usr/lib/php/modules/{enchant,gd,imap,intl,ldap,mcrypt,mssql,odbc,pdo_odbc,pgsql,pdo_pgsql,pspell,snmp,sqlite3,pdo_sqlite,tidy,xsl}.so
	# remove empty directory
	rmdir ${pkgdir}/usr/include/php/include
	# fix broken link
	ln -sf phar.phar ${pkgdir}/usr/bin/phar
}

package_php-cgi() {
	pkgdesc='CGI and FCGI SAPI for PHP'
	depends=('php')

	install -D -m755 ${srcdir}/build-cgi/sapi/cgi/php-cgi ${pkgdir}/usr/bin/php-cgi
}

package_php-apache() {
	pkgdesc='Apache SAPI for PHP'
	depends=('php' 'apache')
	backup=('etc/httpd/conf/extra/php5_module.conf')

	install -D -m755 ${srcdir}/build-apache/libs/libphp5.so ${pkgdir}/usr/lib/httpd/modules/libphp5.so
	install -D -m644 ${srcdir}/apache.conf ${pkgdir}/etc/httpd/conf/extra/php5_module.conf
}

package_php-fpm() {
	pkgdesc='FastCGI Process Manager for PHP'
	depends=('php' 'systemd')
	backup=('etc/php/php-fpm.conf')
	install='php-fpm.install'

	install -D -m755 ${srcdir}/build-fpm/sapi/fpm/php-fpm ${pkgdir}/usr/bin/php-fpm
	install -D -m644 ${srcdir}/build-fpm/sapi/fpm/php-fpm.8 ${pkgdir}/usr/share/man/man8/php-fpm.8
	install -D -m644 ${srcdir}/build-fpm/sapi/fpm/php-fpm.conf ${pkgdir}/etc/php/php-fpm.conf
	install -D -m644 ${srcdir}/logrotate.d.php-fpm ${pkgdir}/etc/logrotate.d/php-fpm
	install -d -m755 ${pkgdir}/etc/php/fpm.d
	install -D -m644 ${srcdir}/php-fpm.tmpfiles ${pkgdir}/usr/lib/tmpfiles.d/php-fpm.conf
	install -D -m644 ${srcdir}/php-fpm.service ${pkgdir}/usr/lib/systemd/system/php-fpm.service
}

package_php-embed() {
	pkgdesc='Embedded PHP SAPI library'
	depends=('php')

	install -D -m755 ${srcdir}/build-embed/libs/libphp5.so ${pkgdir}/usr/lib/libphp5.so
	install -D -m644 ${srcdir}/${pkgbase}-${pkgver}/sapi/embed/php_embed.h ${pkgdir}/usr/include/php/sapi/embed/php_embed.h
}

package_php-phpdbg() {
	pkgdesc='Interactive PHP debugger'
	depends=('php')

	install -D -m755 ${srcdir}/build-phpdbg/sapi/phpdbg/phpdbg ${pkgdir}/usr/bin/phpdbg
}

package_php-pear() {
	pkgdesc='PHP Extension and Application Repository'
	depends=('php')
	backup=('etc/php/pear.conf')

	cd ${srcdir}/build-pear
	make install-pear INSTALL_ROOT=${pkgdir}
	rm -rf ${pkgdir}/usr/share/pear/.{channels,depdb,depdblock,filemap,lock,registry}
}

package_php-enchant() {
	pkgdesc='enchant module for PHP'
	depends=('php' 'enchant')

	install -D -m755 ${srcdir}/build-php/modules/enchant.so ${pkgdir}/usr/lib/php/modules/enchant.so
}

package_php-gd() {
	pkgdesc='gd module for PHP'
	depends=('php' 'gd')

	install -D -m755 ${srcdir}/build-php/modules/gd.so ${pkgdir}/usr/lib/php/modules/gd.so
}

package_php-imap() {
	pkgdesc='imap module for PHP'
	depends=('php' 'c-client')

	install -D -m755 ${srcdir}/build-php/modules/imap.so ${pkgdir}/usr/lib/php/modules/imap.so
}

package_php-intl() {
	pkgdesc='intl module for PHP'
	depends=('php' 'icu')

	install -D -m755 ${srcdir}/build-php/modules/intl.so ${pkgdir}/usr/lib/php/modules/intl.so
}

package_php-ldap() {
	pkgdesc='ldap module for PHP'
	depends=('php' 'libldap')

	install -D -m755 ${srcdir}/build-php/modules/ldap.so ${pkgdir}/usr/lib/php/modules/ldap.so
}

package_php-mcrypt() {
	pkgdesc='mcrypt module for PHP'
	depends=('php' 'libmcrypt' 'libltdl')

	install -D -m755 ${srcdir}/build-php/modules/mcrypt.so ${pkgdir}/usr/lib/php/modules/mcrypt.so
}

package_php-mssql() {
	pkgdesc='mssql module for PHP'
	depends=('php' 'freetds')

	install -D -m755 ${srcdir}/build-php/modules/mssql.so ${pkgdir}/usr/lib/php/modules/mssql.so
}

package_php-odbc() {
	pkgdesc='ODBC modules for PHP'
	depends=('php' 'unixodbc')

	install -D -m755 ${srcdir}/build-php/modules/odbc.so ${pkgdir}/usr/lib/php/modules/odbc.so
	install -D -m755 ${srcdir}/build-php/modules/pdo_odbc.so ${pkgdir}/usr/lib/php/modules/pdo_odbc.so
}

package_php-pgsql() {
	pkgdesc='PostgreSQL modules for PHP'
	depends=('php' 'postgresql-libs')

	install -D -m755 ${srcdir}/build-php/modules/pgsql.so ${pkgdir}/usr/lib/php/modules/pgsql.so
	install -D -m755 ${srcdir}/build-php/modules/pdo_pgsql.so ${pkgdir}/usr/lib/php/modules/pdo_pgsql.so
}

package_php-pspell() {
	pkgdesc='pspell module for PHP'
	depends=('php' 'aspell')

	install -D -m755 ${srcdir}/build-php/modules/pspell.so ${pkgdir}/usr/lib/php/modules/pspell.so
}

package_php-snmp() {
	pkgdesc='snmp module for PHP'
	depends=('php' 'net-snmp')

	install -D -m755 ${srcdir}/build-php/modules/snmp.so ${pkgdir}/usr/lib/php/modules/snmp.so
}

package_php-sqlite() {
	pkgdesc='sqlite module for PHP'
	depends=('php' 'sqlite')

	install -D -m755 ${srcdir}/build-php/modules/sqlite3.so ${pkgdir}/usr/lib/php/modules/sqlite3.so
	install -D -m755 ${srcdir}/build-php/modules/pdo_sqlite.so ${pkgdir}/usr/lib/php/modules/pdo_sqlite.so
}

package_php-tidy() {
	pkgdesc='tidy module for PHP'
	depends=('php' 'tidyhtml')

	install -D -m755 ${srcdir}/build-php/modules/tidy.so ${pkgdir}/usr/lib/php/modules/tidy.so
}

package_php-xsl() {
	pkgdesc='xsl module for PHP'
	depends=('php' 'libxslt')

	install -D -m755 ${srcdir}/build-php/modules/xsl.so ${pkgdir}/usr/lib/php/modules/xsl.so
}

