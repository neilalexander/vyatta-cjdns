#!/usr/bin/make -f

PACKAGE=vyatta-cjdns

DESTDIR=/opt/vyatta

TEMPLATECFGDIR=$DESTDIR/share/vyatta-cfg/template
TEMPLATEOPDIR=$DESTDIR/share/vyatta-op/template
SBINDIR=$DESTDIR/sbin/

BUILDDIR_CJD=/tmp/cjdns

CFLAGS:=$(shell dpkg-buildflags --get CFLAGS)
LDFLAGS:=$(shell dpkg-buildflags --get LDFLAGS)

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
	git clone https://github.com/cjdelisle/cjdns $(BUILDDIR_CJD)
	touch /tmp/cjdns/cjdroute

cjdnsbuild:
	cd $(BUILDDIR_CJD) && TARGET_ARCH='mips' CROSS_COMPILE='mipsel-linux-gnu-' Seccomp_NO=1 ./cross-do

cjdnsstage:
	cp $(BUILDDIR_CJD)/cjdroute sbin/cjdroute

clean:
	rm -rf sbin/cjdroute
	rm -rf $(BUILDDIR_CJD)

distclean:	clean
