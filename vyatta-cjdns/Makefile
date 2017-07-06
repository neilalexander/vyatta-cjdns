#!/usr/bin/make -f

PACKAGE=vyatta-cjdns

DESTDIR=/opt/vyatta

TEMPLATECFGDIR=$DESTDIR/share/vyatta-cfg/template
TEMPLATEOPDIR=$DESTDIR/share/vyatta-op/template
SBINDIR=$DESTDIR/sbin/

BUILDDIR_CJD=/tmp/cjdns

PREFIX?='mipsel-linux-gnu-'
PKGARCH?=$(shell echo $PREFIX | cut -d '-' -f 1 -)

CFLAGS:=$(shell dpkg-buildflags --get CFLAGS)
LDFLAGS:=$(shell dpkg-buildflags --get LDFLAGS)

BRANCH?='master'

all: cross

cross: clean build-cross package cleanpackage
native: clean build-native package cleanpackage

build-cross: cjdnsclone cjdnsbuild-cross cjdnsstage
build-native: cjdnsclone cjdnsbuild-native cjdnsstage

package:
	sed -i "s/^Architecture: .*/Architecture: $(PKGARCH)/" debian/control
	tar czvf ../data.tar.gz opt
	cd debian && tar czvf ../../control.tar.gz *
	echo 2.0 > ../debian-binary
	ar -r ../$(PACKAGE).deb ../debian-binary ../control.tar.gz ../data.tar.gz

cleanpackage:
	rm ../debian-binary
	rm ../control.tar.gz
	rm ../data.tar.gz

cjdnsclone:
	git clone -b $(BRANCH) https://github.com/cjdelisle/cjdns $(BUILDDIR_CJD)
	touch /tmp/cjdns/cjdroute

cjdnsbuild-cross:
	cd $(BUILDDIR_CJD) && TARGET_ARCH='mips' CROSS_COMPILE=$(PREFIX) CROSS_CFLAGS="-mips32r2" Seccomp_NO=1 ./cross-do

cjdnsbuild-native:
	cd $(BUILDDIR_CJD) && ./do

cjdnsstage:
	cp $(BUILDDIR_CJD)/cjdroute opt/vyatta/sbin/cjdroute

clean:
	rm -rf sbin/cjdroute
	rm -rf $(BUILDDIR_CJD)

distclean:	clean
