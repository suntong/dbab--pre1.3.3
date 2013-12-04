= dbab -- dnsmasq based ad blocking

== SYNOPSIS

    pixelserv &

== DESCRIPTION

Ad blocking by using DNSmasq + Pixelserv.

- Block accessing to the ad sites from the DNS level
- Leave the web pages intact, without any pattern matching and string substitution.
- All ads will be replaced by a 1×1 pixel gif image served locally by the Pixelserv server.

== ALTERNATIVES

People may also use browsers' adblock-plus extension to block ads, but fewer think over how it works internally. Here is an overview of Adblock Plus from a thousand mile high [1] -- whenever the browser needs to load something, the extension kicks in and do a thorough pattern matching of all known ad urls using regular expressions, then hectically replace all found ad urls with something else. This is done on every page, every load, and every component of the web page, using JavaScript. Thus it is by nature slow and CPU intensive, at least inefficient. There are other alternatives to this, e.g., privoxy, but the concepts are the same.

[1] http://adblockplus.org/en/faq_internal

== AUTHOR

Copyright: 2013 Tong Sun <suntong001@users.sourceforge.net>  
License: BSD-3-Clause

The pixelserv was downloaded from  
 http://proxytunnel.sourceforge.net/files/pixelserv.pl.txt  
Originally wrote by Piet Wintjens.
