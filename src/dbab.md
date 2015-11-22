# Dbab From Start To Finish

The following introduction is for `dbab` at or over [version 1.3.1](https://github.com/suntong/dbab/releases/tag/v1.3.1), which is incompatible with previous version (1.2.x) as the configuration files have been renamed. The latest version of this article is [available here](https://github.com/suntong/dbab/blob/master/src/dbab.md).

<a name="advantages"/>

## Dbab Advantages

Before dipping into the following details, here are the advantages of using `dbab` (Dnsmasq-Based Ad-Blocking).

First of all, *why this is the best method for ad blocking?* Because all other filter based solutions (privoxy, Adblock-Plus, etc) are CPU intensive because of a large quantity of ad urls and page contents need to be pattern matched, and using regular expressions matching is expensive. Adblock Plus, the easiest choice, is actually the worst choice because it is JavaScript based, and it is the slowest. Furthermore, all these method will more or less alter the rendered web page, to remove the ads. This will be even slower, and might cause side effects as well. 

The `dbab` is however, using an entirely different approach for ad blocking. It's advantages are:

- **Work at the DNS level**. Leave the web pages intact, without any pattern matching, string substitution, and/or html elements replacing.
- **Work for your mobile devices as well**. Were you previously in the dilemma of choosing ads free or slow response for your mobile devices (iphone, ipad, etc)? Now you don't. You don't need to install any thing to your mobile devices for them to enjoy the ad-free browsing experience. Moreover, their browsing speed will increase dramatically on revisited pages/images. 
- **Serve instantly**. All ads will be replaced by a `1x1` pixel gif image locally served by the Pixelserv server.
- **Maintenance free**. You don't need to maintain the list of ad sites yourself. The block list can be downloaded from pgl.yoyo.org periodically. If you don't like some of the entries there, you can define your local tweaking that filters them out.
- **Easily customized**. It's trivial to add your own entries to the ad blocking list if the existing ones are not enough for you.


<a name="static"/>

## Static IP

Now let's start. First, if you haven't done [switching from dynamic IP to static IP](http://sfxpt.wordpress.com/2014/05/11/use-dbab-under-ubuntu-14-04-trusty/) yet, check it out first for how to

- configure the static IP, and
- add a second static IP address

and check out [why to do that](http://sfxpt.wordpress.com/2011/02/21/the-best-ad-blocking-method/#Pixelserv_server_IP_address) as well if you want. 

Here is a recap what I did to configure my machine with the `192.168.2.102` static IP and a second one of `192.168.2.101`:

```bash
cat << EOF > /etc/network/interfaces
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
allow-hotplug eth0

# Use static IP instead of dhcp
iface eth0 inet static
        address 192.168.2.102
        network 192.168.2.0
        netmask 255.255.255.0
        broadcast 192.168.2.255
        gateway 192.168.2.1
        # add a 2nd ip address
        post-up ip addr add dev eth0 192.168.2.101/24
        pre-down ip addr del dev eth0 192.168.2.101/24
EOF

/etc/init.d/network-manager restart
/etc/init.d/networking restart

% ip addr show dev eth0
...
    inet 192.168.2.102/24 brd 192.168.2.255 scope global eth0
...
    inet 192.168.2.101/24 scope global secondary eth0
...

```

For details and troubleshooting, refer back to the above 
[switching from dynamic IP to static IP](http://sfxpt.wordpress.com/2014/05/11/use-dbab-under-ubuntu-14-04-trusty/) document. 

<a name="plan"/>

## The Plan

Once we have our second IP address, the reset of the steps are:

0. [Install & configure DNSmasq](http://sfxpt.wordpress.com/2013/11/30/dnsmasq-installation-configuration-5/).
0. Remove all existing ad blocking tools if you have any.
0. Stop your local web server temporarily if you have any.
0. Before installation `dbab`, go and visit some websites which have ads on their pages such as "yahoo", "abcnews" or anything, then
0. Install & configure the `dbab` package.
0. Restart your local web server if you have any.
0. Now, visit those pages again in different tabs to see if the ads are removed :-).
0. Install [`squid`](http://en.wikipedia.org/wiki/Squid_(software)) caching server, nothing unusual about that.
0. [Setup](https://sfxpt.wordpress.com/2015/11/21/use-new-dbab-to-set-proxy-automatically-3/) [auto proxy](http://sfxpt.wordpress.com/2014/11/23/the-secret-behind-the-auto-proxy-setting/) for everyone and every tool to use the ads-blocking web caching server.

That shall be it. Mission accomplished.

Details to follow. But please be warned, as there are so many pieces tied together, and thus so many things to configure, the following steps are long. So be warned and be prepared. 


<a name="install"/>

## Install & Configure DNSmasq and Dbab

To install DNSmasq and Dbab

```
% apt-get update 
% apt-get install dnsmasq
% apt-get install dbab
```

<a name="configure-dnsmasq"/>

### Configure DNSmasq

To configure DNSmasq:

	cp /usr/share/doc/dbab/dbab-dnsmasq.service.conf /etc/dnsmasq.d
	cp /usr/share/doc/dbab/dbab-dnsmasq.intranet.conf /etc/dnsmasq.d

The `dbab-dnsmasq.service.conf` provides basic `dnsmasq` service configuration. It's content is pretty standard and consistent across all installations, so you don't need to make any changes to it. The `dbab-dnsmasq.intranet.conf` however, reflects how exactly your intranet is configured. What provided is just a boilerplate, of which every content should be customized. I.e., from the below listing, we can see that the ISP DNS server address, the dhcp lease range, the local-net domain name and the dhcp hosts should all be customized. Edit `/etc/dnsmasq.d/dbab-dnsmasq.intranet.conf` to reflect your true intranet  configuration.

	# == DNS from ISP
	server=192.168.2.1
	
	# == Dhcp lease (start,end,leasetime)
	# supply the range of addresses available for lease and optionally
	# a lease time. If you have more than one network, you will need to
	# repeat this for each network on which you want to supply DHCP
	# service.
	dhcp-range=192.168.2.1,192.168.2.80,48h
	
	# == Domain for dnsmasq. this is optional, but if it is set, it
	# does the following things.
	# 1) Allows DHCP hosts to have fully qualified domain names, as long
	#     as the domain part matches this setting.
	# 2) Sets the "domain" DHCP option thereby potentially setting the
	#    domain of all systems configured by DHCP
	# 3) Provides the domain part for "expand-hosts"
	domain=EXAMPLE.ORG
	
	# == Dhcp hosts. 
	# dhcp-host=00:28:58:3A:EB:A1,192.168.2.20,computer2,infinite
	#          ^                 ^            ^         ^
	#          MAC               IP Address   hostname  lease time
	# E.g.,
	#dhcp-host=00:16:3e:00:00:01,192.168.0.81,kvm1,8h
	#dhcp-host=00:16:3e:00:00:02,192.168.0.82,kvm2,8h

Now test it:

```bash
dig @192.168.2.1 google.ca

/etc/init.d/dnsmasq restart

dig @192.168.2.102 google.ca

echo 'nameserver 192.168.2.102' > /etc/resolv.conf

dig google.ca
```

This is heavily-simplified version. For details and troubleshooting refer to:

- [Providing DHCP and DNS services with DNSMasq](http://sfxpt.wordpress.com/2011/02/06/providing-dhcp-and-dns-services-with-dnsmasq/)
- [DNSmasq Installation & Configuration](http://sfxpt.wordpress.com/2013/11/30/dnsmasq-installation-configuration-5/)

<a name="configure-dbab"/>

#### Faq: dnsmasq: setting capabilities failed

If for any reason that you test `dbab` under docker and you get the following error when starting `dnsmasq` (say with `service dnsmasq start`):

	% service dnsmasq start 
	[....] Starting DNS forwarder and DHCP server: dnsmasq
	dnsmasq: setting capabilities failed: Operation not permitted
	 failed!

The fix is to tell dnsmasq to run as root by adding `user=root` to `/etc/dnsmasq.conf`:

```bash
cp /etc/dnsmasq.conf /tmp
sed -i '/^#user=/s/$/\nuser=root/' /etc/dnsmasq.conf
diff -wU1 /tmp/dnsmasq.conf /etc/dnsmasq.conf

# then
service dnsmasq start
```

Ref: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=514214


### Configure Dbab

The Dbab comes with a simple local configure using the [second static IP address](http://sfxpt.wordpress.com/2014/05/11/use-dbab-under-ubuntu-14-04-trusty/). To configure `dbab` to work with a local web server:

0. Stop `dbab-svr` service
0. Change the IP address that dbab uses to the second IP address
0. Start `dbab-svr` service
0. Start your local web server again if you have any. You may need to limit its listening port from `0.0.0.0` to your first static IP as well if necessary. 

In details, do the following as root, again assuming that the server's own IP address is `192.168.2.102`, and its second IP is `192.168.2.101`. The second IP will be used for the `dbab` service (WPAD & pixelserv).

```bash
# (run the following as root)

# stop dbab service
/etc/init.d/dbab stop

# use the second IP for dbab-svr to listens on
ip -f inet addr show eth0 | awk '/inet /{print $2}' | sed 's|/.*$||; 1d' | sudo tee /etc/dbab/dbab.addr
# verify its content before moving on
cat /etc/dbab/dbab.addr
# if it is not what you intent it to be, correct it with your text editor
  # or, set it manually (with a different IP address)
  echo 192.168.2.101 | sudo tee /etc/dbab/dbab.addr

# update ad blocking list with the second IP address 
/usr/sbin/dbab-get-list
/usr/sbin/dbab-add-list

# OPTIONAL! do the following only if you have squid caching server
# and you want to enable automatic WPAD service
hostname | tee /etc/dbab/dbab.proxy
  # NB, if your squid caching server is on a different server, do this instead
  echo my_squid_server_name | tee /etc/dbab/dbab.proxy
# then, 
/usr/sbin/dhcp-add-wpad
# Again verify everything here before moving on because script might not be
# 100% time correct. Manually tweaking is inevitable sometimes.

# restart DNS & DHCP
/etc/init.d/dnsmasq restart

# re-start dbab service
/etc/init.d/dbab start

# re-start your local web server again if you have any

# optional, only when dbab will not auto start on boot up
update-rc.d dbab defaults

```

That's it. We're done.

#### Faq: How to blacklist those bad sites?

All these started because there were one time that the top of google hits are often crammed with rubbish sites. I.e., those sites that contains nothing but key words merely to be listed on top of google hits. These sites are called content-farming sites, and goolge has been constantly fighting with them (Google's Farmer Update at the end of February, 2011):

> "So-called content farms such as Demand Media and Associated Content, both routinely vilified for churning out shabbily produced, keyword-loaded content that often secured top listings at Google, were penalized severely." [1]

[1] http://www.websitemagazine.com/content/blogs/posts/pages/crop-devastation-google-s-farmer-update-retools-rankings.aspx

Yet, there are still many content-farming sites that fall through the crack or revamp again. I was so annoyed that, instead of waiting for google to deal with them, I took the matter into my own hand. Here is the updated version that makes use the `dbab` package to block them:

First, gather a list of those rubish sites:

	cat >> /etc/dbab/dbab.list+

The result will look something like this:

	$ cat /etc/dbab/dbab.list+
	dl4all.com
	filestube.com
	terapdf.com
	101com.com

Then, convert the list to be used by DNSmasq:

	/usr/sbin/dbab-add-list

Those bad sites are now blocked by DNSmasq, after restarting it:

    /etc/init.d/dnsmasq restart

That's it. Next time if you accidentally click into those sites,
You will see a blank page, which loads instantly, with the
following as the page title:

    (GIF Image, 1x1 pixels)

Then you know you've stumbled into sites that you should have avoided.


#### Faq: How to whitelist some sites?

First see what exactly was listed in the pgl.yoyo.org list. E.g., to enable `www.googleadservices.com`, merely putting `www.googleadservices.com` into `etc/dbab/dbab.list-` won't help, because:

	$ grep googleadservices /etc/dnsmasq.d/dbab.*
	address=/googleadservices.com/127.0.0.1

I.e., we should put in `googleadservices.com` instead of `www.googleadservices.com`.

Now suppose we need to whitelist `googleadservices.com` and `urlcash.net`, here is how to do:

	echo 'googleadservices.com' > /etc/dbab/dbab.list-
	echo 'urlcash.net' >> /etc/dbab/dbab.list-

	/usr/sbin/dbab-get-list
	grep googleadservices /etc/dnsmasq.d/dbab.*

	service dnsmasq restart 

	dig www.googleadservices.com

It should show real IP instead of `127.0.0.1`.


## Switching Over to DNSmasq Service

To make the above changed configuration take effect, `dnsmasq` must be
restarted (because sending SIGHUP to the dnsmasq process will only cause it
to empty its cache and then re-load /etc/hosts and /etc/resolv.conf):

    /etc/init.d/dnsmasq restart

But before doing that, we need to disable (DSL) router's dhcp and dns
services, because (DSL) router would normally act as both dhchp and dns server
for the most cases. if I dedicate a dnsmasq server for both dhcp and dns
servers, I have to disable DHCP on my router so only my own dnsmasq server
responds to DHCP requests. For DNS, the DHCP response can give the IP
address of the DNS for the clients to use.

Having restarted the `dnsmasq` service, we still can't test anything about DNS leases because `dnsmasq` doesn't return results for dns query until after it has actually served out the address. So we can't prove anything until after a DHCP/DNS request is made.

Again, for details and troubleshooting refer to:

- [The Best Ad Blocking Method](http://sfxpt.wordpress.com/2011/02/21/the-best-ad-blocking-method/)
- [The Best Ad Blocking Method in a Package](http://sfxpt.wordpress.com/2014/01/05/the-best-ad-blocking-method-in-a-package/)
- [Use dbab under Ubuntu 14.04 Trusty](http://sfxpt.wordpress.com/2014/05/11/use-dbab-under-ubuntu-14-04-trusty/)

<a name="squid"/>

## Local Caching Server

Now it is time to make it easy for anyone visiting your home to enjoy you fast local `squid` caching server. Let's continue on that trend to [auto proxy setting](http://sfxpt.wordpress.com/2014/11/23/the-secret-behind-the-auto-proxy-setting/). I.e., [DNSMasq gets DHCP and DNS together](http://sfxpt.wordpress.com/2011/02/06/providing-dhcp-and-dns-services-with-dnsmasq/), and [the dbab](http://sfxpt.wordpress.com/2014/01/05/the-best-ad-blocking-method-in-a-package/) brings them both and [ad blocking](http://sfxpt.wordpress.com/2011/02/21/the-best-ad-blocking-method/) together, and now let's move a step further to bring [`squid`](http://en.wikipedia.org/wiki/Squid_(software)) and  [auto proxy setting](http://sfxpt.wordpress.com/2014/11/23/the-secret-behind-the-auto-proxy-setting/) into the picture and into the harmony. 

### Strategy

To recap, we need a dedicated server in the SOHO environment for

- DHCP, & DNS using [DNSmasq](http://sfxpt.wordpress.com/2013/11/30/dnsmasq-installation-configuration-5/), which we have just installed.
- a caching server/proxy using `squid`, which we will install next.
- [use `dbab` to provide WPAD & pixelserv service](https://sfxpt.wordpress.com/2015/11/21/use-new-dbab-to-set-proxy-automatically-3/) and join them all together

All of them are hosted on a single machine. This is a typical and reasonable configuration, because even with all above, the machine does not need to be a powerful or even a fast one. Mine is a Pentium 5, with 4G of RAM and 300G of disk space, and have a web server, a time server, a printing server, an email server and a SSH server installed as well, along with the DHCP, DNS & web caching server, and it has more than enough power to handle everything. 

So install the `squid` caching server on this dedicated SOHO server as normal, and start it. The `dbab` should have already properly configured to use it. See above listing for how *"to enable automatic WPAD service"*.

## Verify

To check ad blocking, revisit in new tabs those pages you just visited that's full of ads, and compare the differences, or check out the following urls, which are automatically blocked by the `dbab-get-list` command:

http://actualdeals.com/  
http://ad.about.com/  
http://ad.abcnews.com/  
http://ad.abcnews.com/anything/else  

To check your automatic proxy setting, use:

    $ curl http://wpad/wpad.dat
    function FindProxyForURL(url, host) { return "PROXY mysohosvr:3128; DIRECT"; }

The `http://wpad/wpad.dat` will always be the same regardless how your servers are called, but `mysohosvr` shall be the real name of your squid caching server. 

To check your automatic proxy results, first [set up your browser to use WPAD](http://goo.gl/9uofLX#heading=h.7wr0f68pdads), then on your SOHO server do the following before visiting any pages:

    tail -f /var/log/squid3/access.log

If the places you are visiting show up in the access log, then everything is working. Now fire up your iphone or ipad to visit some sites. As long as your iphone or ipad is using WIFI from your SOHO network, their visit will be cached as well. Or at least so I read. Check the access log to verify. As for Android, sorry, while iphone or ipad are playing by the rules to set proxy automatically from WPAD, Android isn't. You have to set its proxy manually. Visit some pages with some very-slow-loading pictures, and they visit them again, the picture loading speed will be dramatically faster, especially if your wireless device is not super fast (like mine).

If AOK, you may want to setup a cron job to update the block list on a weekly/monthly basis. E.g.:

    ln -s /usr/sbin/dbab-get-list /etc/cron.monthly/

