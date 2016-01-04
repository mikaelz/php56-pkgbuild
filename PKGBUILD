pkgbase=php56
pkgname=(
    'php56-standalone'
)
pkgver=5.6.16
pkgrel=1
prefix=/opt/php56
fpmuser=alex
arch=('i686' 'x86_64')
license=('PHP')
url='http://www.php.net'
makedepends=('apache' 'imap' 'postgresql-libs' 'libldap' 'postfix' 'libvpx'
             'sqlite' 'unixodbc' 'net-snmp' 'libzip' 'enchant' 'file' 'freetds'
             'libmcrypt' 'tidyhtml' 'aspell' 'libltdl' 'libpng' 'libjpeg' 'icu'
             'curl' 'libxslt' 'openssl' 'bzip2' 'db' 'gmp' 'freetype2' 'systemd')
source=(
    "http://www.php.net/distributions/${pkgbase%56}-${pkgver}.tar.bz2"
    'php-fpm.service' 
    'php-fpm.tmpfiles'
)

prepare() {
	cd ${srcdir}/${pkgbase%56}-${pkgver}
	cpus=`cat /proc/cpuinfo | grep processor | wc -l`
}

build() {
	local _phpconfig="--srcdir=../${pkgbase%56}-${pkgver} \
		--config-cache \
		--prefix=${prefix}/usr \
		--sbindir=${prefix}/usr/bin \
		--sysconfdir=${prefix}/etc/php \
		--localstatedir=/var \
		--with-layout=GNU \
		--with-config-file-path=${prefix}/etc/php \
		--with-config-file-scan-dir=${prefix}/etc/php/conf.d \
		--disable-rpath \
		--mandir=${prefix}/usr/share/man \
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
		--with-gd=shared \
		--with-gdbm \
		--with-gettext=shared \
		--with-gmp=shared \
		--with-iconv=shared \
		--with-icu-dir=/usr \
		--with-imap-ssl \
		--with-imap=shared \
		--with-jpeg-dir=/usr \
		--with-vpx-dir=/usr \
		--with-ldap=shared \
		--with-ldap-sasl \
		--with-mcrypt=shared \
		--with-mhash \
		--with-mssql=shared \
		--with-mysql-sock=/var/run/mysqld/mysqld.sock \
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
		--with-kerberos \
		"

	EXTENSION_DIR=/opt/php56/usr/lib/php/modules
	export EXTENSION_DIR
	PEAR_INSTALLDIR=/opt/php56/usr/share/pear
	export PEAR_INSTALLDIR

	cd ${srcdir}/${pkgbase%56}-${pkgver}

	# php
	if [ ! -d ${srcdir}/build-php ]; then
        mkdir ${srcdir}/build-php
    fi
	cd ${srcdir}/build-php
	ln -sf ../${pkgbase%56}-${pkgver}/configure
	./configure ${_phpconfig} \
		--disable-cgi \
		--with-readline \
		--enable-pcntl \
		${_phpextensions}
	make -j${cpus}

	# apache
	cp -a ${srcdir}/build-php ${srcdir}/build-apache
	cd ${srcdir}/build-apache
	./configure ${_phpconfig} \
		--disable-cli \
		--with-apxs2 \
		${_phpextensions}
	make -j${cpus}

	# fpm
	cp -a ${srcdir}/build-php ${srcdir}/build-fpm
	cd ${srcdir}/build-fpm
	./configure ${_phpconfig} \
		--disable-cli \
		--enable-fpm \
		--with-fpm-user=${fpmuser} \
		--with-fpm-group=${fpmuser} \
		${_phpextensions}
	make -j${cpus}

	# pear
	cp -a ${srcdir}/build-php ${srcdir}/build-pear
	cd ${srcdir}/build-pear
	./configure ${_phpconfig} \
		--disable-cgi \
		--with-readline \
		--enable-pcntl \
		--with-pear \
		${_phpextensions}
	make -j${cpus}
}

# check() {
# 	cd ${srcdir}/build-php
# 	make test
# }

package_php56-standalone() {
	pkgdesc='An HTML-embedded scripting language'
	depends=('pcre' 'libxml2' 'bzip2' 'curl')
	install='php.install'
	backup=(
        'etc/php/php.ini'
        'etc/httpd/conf/extra/php5_module.conf'
        'etc/php/php-fpm.conf'
        'etc/php/pear.conf'
    )

	cd ${srcdir}/build-php
	make -j${cpus} INSTALL_ROOT=${pkgdir} install
	install -d -m755 ${pkgdir}${prefix}/usr/share/pear
	# install php.ini
	install -D -m644 ${srcdir}/${pkgbase%56}-${pkgver}/php.ini-production ${pkgdir}${prefix}/etc/php/php.ini
	install -d -m755 ${pkgdir}${prefix}/etc/php/conf.d/

	# remove static modules
	rm -f ${pkgdir}${prefix}/usr/lib/php/modules/*.a
	# remove modules provided by sub packages
	rm -f ${pkgdir}${prefix}/usr/lib/php/modules/{enchant,gd,intl,ldap,mcrypt,mssql,odbc,pdo_odbc,pgsql,pdo_pgsql,pspell,snmp,sqlite3,pdo_sqlite,tidy,xsl}.so
	
	# apache
	install -D -m755 ${srcdir}/build-apache/libs/libphp5.so ${pkgdir}${prefix}/usr/lib/httpd/modules/libphp5.so
	
	# fpm
	install -D -m755 ${srcdir}/build-fpm/sapi/fpm/php-fpm ${pkgdir}${prefix}/usr/bin/php-fpm
	install -D -m644 ${srcdir}/build-fpm/sapi/fpm/php-fpm.8 ${pkgdir}${prefix}/usr/share/man/man8/php-fpm.8
	install -D -m644 ${srcdir}/build-fpm/sapi/fpm/php-fpm.conf ${pkgdir}${prefix}/etc/php/php-fpm.conf
	install -d -m755 ${pkgdir}${prefix}/etc/php/fpm.d
	install -D -m644 ${srcdir}/php-fpm.tmpfiles ${pkgdir}${prefix}/usr/lib/tmpfiles.d/php56-fpm.conf
	install -D -m644 ${srcdir}/php-fpm.service ${pkgdir}${prefix}/usr/lib/systemd/system/php56-fpm.service
	
	cd ${srcdir}/build-pear
	make install-pear INSTALL_ROOT=${pkgdir}
	rm -rf ${pkgdir}/usr/share/pear/.{channels,depdb,depdblock,filemap,lock,registry}
}

md5sums=(
    '765b164b89af1f03ae2fdf39a4e4e1df' # php56
    '50215d7f33da9b0ba32fa6e1451e4bae' # php-fpm.service
    'c60343df74f8e1afb13b084d5c0e47ed' # php-fpm.tmpfiles
)
