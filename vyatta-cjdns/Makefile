#!/usr/bin/make -f
#
# vyatta-cjdns - A cjdns package for Ubiquiti EdgeOS, VyOS and other
# Vyatta-based systems.
# Copyright (C) 2015 Neil Alexander T.
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#

PACKAGE=vyatta-cjdns
RELEASE=0.2

DESTDIR=/opt/vyatta

TEMPLATECFGDIR=$DESTDIR/share/vyatta-cfg/template
TEMPLATEOPDIR=$DESTDIR/share/vyatta-op/template
SBINDIR=$DESTDIR/sbin/

BUILDDIR_CJD=/tmp/cjdns

PREFIX?='mipsel-linux-gnu-'
PKGARCH?=$(shell dpkg --print-architecture)
BUILDNUMBER=$(shell git rev-list v$(RELEASE)..HEAD --count 2>/dev/null || echo 0)
ifeq ($(BUILDNUMBER),0)
VERSION=$(RELEASE)
else
VERSION=$(RELEASE).$(shell echo $(BUILDNUMBER)+1 | bc)
endif

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
	sed -i "s/^Version: .*/Version: $(VERSION)/" debian/control
	tar czvf ../data.tar.gz opt
	cd debian && tar czvf ../../control.tar.gz *
	echo 2.0 > ../debian-binary
	ar -r ../$(PACKAGE)-$(VERSION)-$(PKGARCH).deb ../debian-binary ../control.tar.gz ../data.tar.gz

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
