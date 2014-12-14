INSTALL = /usr/bin/install -c

# Installation directories
prefix = ${DESTDIR}/usr
exec_prefix = ${prefix}
mandir = ${prefix}/share/man/man8
docdir = ${prefix}/share/doc/dbab
bindir = ${exec_prefix}/sbin
etcdir = ${DESTDIR}/etc
astdir = ${etcdir}/dbab

man:
	cat README.md | ronn > assets/dbab-svr.8

clean:
	rm -f assets/dbab-svr.8

install:
	$(INSTALL) -m 755 -d $(bindir)
	$(INSTALL) -m 755 -d $(mandir)
	$(INSTALL) -m 755 -d $(docdir)
	$(INSTALL) -m 755 -d $(etcdir)
	$(INSTALL) -m 755 -d $(etcdir)/init.d
	$(INSTALL) -m 755 -d $(astdir)

	$(INSTALL) -m 755 bin/dbab-get-list $(bindir)
	$(INSTALL) -m 755 bin/dbab-add-list $(bindir)
	$(INSTALL) -m 755 bin/dbab-chk-list $(bindir)
	$(INSTALL) -m 755 bin/dbab-svr $(bindir)
	$(INSTALL) -m 755 bin/dbab $(etcdir)/init.d

	$(INSTALL) -m 644 assets/dbab-svr.8 $(mandir)
	$(INSTALL) -m 644 assets/dbab-add-list.1 $(mandir)
	$(INSTALL) -m 644 assets/dbab-chk-list.1 $(mandir)
	$(INSTALL) -m 644 assets/dbab-get-list.1 $(mandir)
	$(INSTALL) -m 644 assets/dhcp-add-wpad.1 $(mandir)
	$(INSTALL) -m 644 dbab.html $(docdir)

	$(INSTALL) -m 644 assets/dbab.addr $(astdir)
	$(INSTALL) -m 644 assets/dbab.list+ $(astdir)
	$(INSTALL) -m 644 assets/dbab.list- $(astdir)
	$(INSTALL) -m 644 assets/dbab.proxy $(astdir)
