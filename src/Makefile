INSTALL = /usr/bin/install -c

# Installation directories
prefix = ${DESTDIR}/usr
exec_prefix = ${prefix}
mandir = ${prefix}/share/man/man8
bindir = ${exec_prefix}/sbin
etcdir = ${DESTDIR}/etc

man:
	cat README.md | ronn > assets/dbab-pixelserv.8

clean:
	rm -f assets/dbab-pixelserv.8

install:
	$(INSTALL) -m 755 -d $(bindir)
	$(INSTALL) -m 755 -d $(mandir)
	$(INSTALL) -m 755 -d $(etcdir)

	$(INSTALL) -m 755 bin/dbab-pixelserv $(bindir)
	$(INSTALL) -m 755 bin/dbab-get-list $(bindir)
	$(INSTALL) -m 755 bin/dbab-add-list $(bindir)
	$(INSTALL) -m 755 bin/dbab-chk-list $(bindir)

	$(INSTALL) -m 644 assets/dbab-pixelserv.8 $(mandir)
#	$(INSTALL) -m 644 assets/dbab-get-list.8 $(mandir)
#	$(INSTALL) -m 644 assets/dbab-add-list.8 $(mandir)
#	$(INSTALL) -m 644 assets/dbab-chk-list.8 $(mandir)

	$(INSTALL) -m 644 assets/dbab.addr $(etcdir)
	$(INSTALL) -m 644 assets/dbab.list+ $(etcdir)
	$(INSTALL) -m 644 assets/dbab.list- $(etcdir)
