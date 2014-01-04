# dbab(8) -- dnsmasq based ad blocking

## SYNOPSIS

    # start dbab-pixelserv server
	/etc/init.d/dbab-service start

    # stop dbab-pixelserv server
	/etc/init.d/dbab-service stop

    # get/update ad blocking list
	/usr/sbin/dbab-get-list

	# add your own to the ad blocking list
	/usr/sbin/dbab-add-list


## DESCRIPTION

Ad blocking by using DNSmasq + Pixelserv -- block accessing to the ad sites from the DNS level. I.e., the DNS server is smart enough to know which sites are ad-sites and block access to them right there. No more user space extensive pattern matching necessary at all. 


## ALTERNATIVES

People may also use browsers' adblock-plus extension to block ads, but fewer think over how it works internally. Here is an overview of Adblock Plus from a thousand mile high [1] -- whenever the browser needs to load something, the extension kicks in and do a thorough pattern matching of all known ad urls using regular expressions, then hectically replace all found ad urls with something else. This is done on every page, every load, and every component of the web page, using JavaScript. Thus it is by nature slow and CPU intensive, at least inefficient. There are other alternatives to this, e.g., privoxy, but the concepts are the same.

[1] http://adblockplus.org/en/faq_internal

## ADVANTAGES

Comparing to other ad-blocking efforts, `dbab` will be super light. Only a few operations are enough to determine and stop the ads. No heavy-lifting (using CPU intensive URL pattern matching) necessary. Thus it will be lighting fast as well. 

The advantages of using `dbab` are:

- **Work at the DNS level**. Leave the web pages intact, without any pattern matching, string substitution, and/or html elements replacing.
- **Serve instantly**. All ads will be replaced by a 1x1 pixel gif image served locally by the `dbab-pixelserv` server.
- **Maintenance free**. You don't need to maintain the list of ad sites yourself. The block list can be downloaded from pgl.yoyo.org periodically. If you don't like some of the entries there, you can define your local tweaking that filter them out.

## DBAB-PIXELSERV

The `dbab-pixelserv` is a super minimal web server, it's one and only purpose is serving a 1x1 pixel transparent gif file. By default it listens on localhost. If you also have a normal webserver on your DNS server, you can

- add a second IP address to your DNSmasq server using a virtual interface, or
- use the virtual hosts option of your local http server to server the 1x1 pixel gif image.

## DBAB-GET-LIST

The `dbab-get-list` is used to get dnsmasq blocking list from pgl.yoyo.org to be used by DNSmasq.

You can run it once, or put it in a cron job so as to update the block list periodically. E.g., to update on a weekly basis:

    ln -s /usr/sbin/dbab-get-list /etc/cron.weekly/

## DBAB-ADD-LIST

You can use `dbab-add-list` to add your own entries to `dnsmasq` blocking list, if the list from pgl.yoyo.org is not sufficient for you. 

## DBAB-CHK-LIST

The `dbab-chk-list` can help you to check if your own list is already covered by pgl.yoyo.org.

## FILES 

* /etc/dbab.addr:  
  The IP address that `dbab-pixelserv` listens on. Defaults to localhost.
  
* /etc/dbab.list-:  
  The entries you want to filter out from the pgl.yoyo.org lists. List sites you still wish to visit there. 

* /etc/dbab.list+:  
  The entries you want to add to blocking list on top of the pgl.yoyo.org list, used by `dbab-add-list`. 


## AUTHOR

Copyright: 2013~2014 Tong Sun `suntong001 at users.sourceforge.net`  
License: BSD-3-Clause

The pixelserv was originally downloaded from  
 http://proxytunnel.sourceforge.net/files/pixelserv.pl.txt  
Wrote by Piet Wintjens, with BSD (no advertising clause) license.
