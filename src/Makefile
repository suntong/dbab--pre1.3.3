INSTALL = /usr/bin/install -c

# Installation directories
prefix = ${DESTDIR}/usr
exec_prefix = ${prefix}
mandir = ${prefix}/share/man
docdir = ${prefix}/share/doc/dbab
bindir = ${exec_prefix}/sbin
libdir = ${DESTDIR}/lib
etcdir = ${DESTDIR}/etc
astdir = ${etcdir}/dbab
ssddir = ${DESTDIR}/lib/systemd/system

# Install the systemd unit file if systemd support was detected.
ifdef HAVE_SYSTEMD
systemdsystemunit_DATA = systemd/dbab.service
endif

man:
	cat README.md | ronn > assets/dbab-svr.8

clean:
	rm -f assets/dbab-svr.8

install:
	$(INSTALL) -m 755 -d $(bindir)
	$(INSTALL) -m 755 -d $(mandir)/man8
	$(INSTALL) -m 755 -d $(docdir)
	$(INSTALL) -m 755 -d $(libdir)/init
	$(INSTALL) -m 755 -d $(etcdir)
	$(INSTALL) -m 755 -d $(etcdir)/init.d
	$(INSTALL) -m 755 -d $(astdir)
	$(INSTALL) -m 755 -d $(ssddir)

	$(INSTALL) -m 755 bin/dbab-get-list $(bindir)
	$(INSTALL) -m 755 bin/dbab-add-list $(bindir)
	$(INSTALL) -m 755 bin/dbab-chk-list $(bindir)
	$(INSTALL) -m 755 bin/dhcp-add-wpad $(bindir)
	$(INSTALL) -m 755 bin/dbab-svr $(bindir)
	$(INSTALL) -m 755 bin/dbab-init-d-script $(libdir)/init
	$(INSTALL) -m 755 bin/dbab $(etcdir)/init.d
	$(INSTALL) -m 644 assets/dbab.service $(ssddir)

	$(INSTALL) -m 644 assets/dbab-svr.8 $(mandir)/man8
	$(INSTALL) -m 644 assets/dbab-add-list.8 $(mandir)/man8
	$(INSTALL) -m 644 assets/dbab-chk-list.8 $(mandir)/man8
	$(INSTALL) -m 644 assets/dbab-get-list.8 $(mandir)/man8
	$(INSTALL) -m 644 assets/dhcp-add-wpad.8 $(mandir)/man8
	$(INSTALL) -m 644 dbab.md $(docdir)

	$(INSTALL) -m 644 assets/dbab.addr $(astdir)
	$(INSTALL) -m 644 assets/dbab.list+ $(astdir)
	$(INSTALL) -m 644 assets/dbab.list- $(astdir)
	$(INSTALL) -m 644 assets/dbab.proxy $(astdir)
