#!/usr/bin/make -f

PACKAGE=vyatta-cjdns

DESTDIR=/opt/vyatta

TEMPLATECFGDIR=$DESTDIR/share/vyatta-cfg/template
TEMPLATEOPDIR=$DESTDIR/share/vyatta-op/template
SBINDIR=$DESTDIR/sbin/

BUILDDIR_CJD=/tmp/cjdns

PREFIX='mipsel-linux-gnu-'

CFLAGS:=$(shell dpkg-buildflags --get CFLAGS)
LDFLAGS:=$(shell dpkg-buildflags --get LDFLAGS)

BRANCH?='master'

all: clean build package cleanpackage

build: cjdnsclone cjdnsbuild cjdnsstage

package:
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

cjdnsbuild:
	cd $(BUILDDIR_CJD) && TARGET_ARCH='mips' CROSS_COMPILE=$(PREFIX) CROSS_CFLAGS="-mips32r2" Seccomp_NO=1 ./cross-do

cjdnsstage:
	cp $(BUILDDIR_CJD)/cjdroute opt/vyatta/sbin/cjdroute

clean:
	rm -rf sbin/cjdroute
	rm -rf $(BUILDDIR_CJD)

distclean:	clean
