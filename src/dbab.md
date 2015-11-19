# Use new dbab to set proxy automatically

In the past, we've covered

- [Providing DHCP and DNS services with DNSMasq](http://sfxpt.wordpress.com/2011/02/06/providing-dhcp-and-dns-services-with-dnsmasq/)
- [DNSmasq Installation & Configuration](http://sfxpt.wordpress.com/2013/11/30/dnsmasq-installation-configuration-5/)
- [The Best Ad Blocking Method](http://sfxpt.wordpress.com/2011/02/21/the-best-ad-blocking-method/)
- [The Best Ad Blocking Method in a Package](http://sfxpt.wordpress.com/2014/01/05/the-best-ad-blocking-method-in-a-package/)
- [Use dbab under Ubuntu 14.04 Trusty](http://sfxpt.wordpress.com/2014/05/11/use-dbab-under-ubuntu-14-04-trusty/)

Now let's continue on that trend to [auto proxy setting](http://sfxpt.wordpress.com/2014/11/23/the-secret-behind-the-auto-proxy-setting/). I.e., [DNSMasq gets DHCP and DNS together](http://sfxpt.wordpress.com/2011/02/06/providing-dhcp-and-dns-services-with-dnsmasq/), and [the dbab](http://sfxpt.wordpress.com/2014/01/05/the-best-ad-blocking-method-in-a-package/) brings them both and [ad blocking](http://sfxpt.wordpress.com/2011/02/21/the-best-ad-blocking-method/) together, and now let's move a step further to bring [`squid`](http://en.wikipedia.org/wiki/Squid_(software)) and  [auto proxy setting](http://sfxpt.wordpress.com/2014/11/23/the-secret-behind-the-auto-proxy-setting/) into the picture, and into the harmony. 

Let's make it easy for anyone visiting your home to enjoy you fast local `squid` caching server, and let's start from all over as if this is the first time you are doing it. But please be warned, as there are so many pieces tied together, and thus so many things to configure, the following steps are long. So be warned and be prepared. 


## Static IP

If you haven't done [switching from dynamic IP to static IP](http://sfxpt.wordpress.com/2014/05/11/use-dbab-under-ubuntu-14-04-trusty/) yet, check it out first for how to

- configure the static IP, and
- add a second static IP address

and check out [here](http://sfxpt.wordpress.com/2011/02/21/the-best-ad-blocking-method/#Pixelserv_server_IP_address) if you want to know why to do that. 

<a name="strategy"/>

## Strategy

The following instructions assume that there is a dedicated server in the SOHO environment for

- DHCP, & DNS using [DNSmasq](http://sfxpt.wordpress.com/2013/11/30/dnsmasq-installation-configuration-5/), 
- a caching server/proxy using squid. 
- and with `dbab` to provide WPAD & pixelserv service and join them all together

All of them are hosted on a single machine. This is a typical and reasonable configuration, because even with all above, the machine does not need to be a powerful or even a fast one. Mine is a Pentium 5, with 8G of RAM and 300G of disk space, and have a web server, a time server, a printing server, an email server and a SSH server installed as well, and it has more than enough power to handle everything. 

Once `dbab` is in the Debian repo, the installation will be so easy that what's important is not the installation but the verification. But before then, use the following PPA for the installation repo instead:

    add-apt-repository -y ppa:suntong001/ppa

Then, 

0. [Switch from dynamic IP to static IP](http://sfxpt.wordpress.com/2014/05/11/use-dbab-under-ubuntu-14-04-trusty/).
0. [Install & configure DNSmasq](http://sfxpt.wordpress.com/2013/11/30/dnsmasq-installation-configuration-5/).
0. Install [`squid`](http://en.wikipedia.org/wiki/Squid_(software)) caching server, nothing unusual about that.
0. Remove all existing ad blocking tools if you have any.
0. Stop your local web server temporarily if you have any.
0. Before installation `dbab`, go and visit some websites which have ads on their pages such as "yahoo", "abcnews" or anything, then
0. Install & configure the `dbab` package.
0. Restart your local web server if you have any.
0. Now, visit those pages again in different tabs to see if the ads are removed :-).

That shall be it. Mission accomplished. 

<a name="install"/>

## Install dbab

To install `dbab`:

```bash
% apt-get update 

% apt-get install dbab
The following extra packages will be installed:
  curl dnsmasq
Suggested packages:
  resolvconf
The following NEW packages will be installed:
  curl dbab dnsmasq
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 145 kB of archives.
After this operation, 482 kB of additional disk space will be used.
Do you want to continue? [Y/n] ...
```

<a name="Configure"/>

## Configure `dbab` to work with a local web server

First of all, [switch from dynamic IP to static IP and add a second static IP address](http://sfxpt.wordpress.com/2014/05/11/use-dbab-under-ubuntu-14-04-trusty/) if you haven't done so.

Once we have our second IP address, the reset is simple:

0. stop dbab-svr service
0. change the IP address that dbab uses to the second IP address
0. start dbab-svr service
0. start your local web server again if you have any

In details, do the following as root, again assuming that the server's own IP address is `192.168.2.100`, and its second IP is `192.168.2.101`. The second IP will be used for the dbab service (WPAD & pixelserv).

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


## Verify

To check ad blocking, revisit in new tabs those pages you just visited that full of ads, and compare the differences, or check out the following urls, which are automatically blocked by the `dbab-get-list` command:

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

    ln -s /usr/sbin/dbab-get-list /etc/cron.weekly

<a name="advantages"/>

## Advantages

Once again, here are the advantages of using dbab (Dnsmasq-Based Ad-Blocking).

First of all, let's recap why this is the best method for ad blocking. All other filter based solution (privoxy, Adblock Plus, etc) are CPU intensive because of a large quantity of ad urls need to be pattern matched, and using regular expressions matching is expensive. Adblock Plus, the easiest choice, is actually the worst choice because it is JavaScript based, and is the slowest. Furthermore, all these method will more or less alter the rendered web page, to remove the ads. This will be even slower, and might cause side effects as well. 

The `dbab` is however, using an entirely different approach for ad blocking. It's advantages are:

- **Work at the DNS level**. Leave the web pages intact, without any pattern matching, string substitution, and/or html elements replacing.
- **Work for your mobile devices as well**. Were you previously in the dilemma of choosing ads free or slow response for your mobile devices (iphone, ipad, etc)? Now you don't. You don't need to install any thing to your mobile devices for them to enjoy the ad-free browsing experience. Moreover, their browsing speed will increase dramatically on revisited pages/images. 
- **Serve instantly**. All ads will be replaced by a `1x1` pixel gif image locally served by the Pixelserv server.
- **Maintenance free**. You don't need to maintain the list of ad sites yourself. The block list can be downloaded from pgl.yoyo.org periodically. If you don't like some of the entries there, you can define your local tweaking that filters them out.
- **Easily customized**. It's trivial to add your own entries to the ad blocking list if the existing ones are not enough for you.

<a name="faq"/>

## Faq

### How to whitelist some sites?

First see what exactly was listed in the pgl.yoyo.org list. E.g., to enable `www.googleadservices.com`, merely putting `www.googleadservices.com` into `etc/dbab/dbab.list-` won't help, because:

	$ grep googleadservices /etc/dnsmasq.d/dbab.*
	address=/googleadservices.com/127.0.0.1

I.e., we should put in `googleadservices.com` instead of `www.googleadservices.com`.

Now suppose we need to whitelist `googleadservices.com` and `urlcash.net`, here is how to do:

	echo 'googleadservices.com' > /etc/dbab/dbab.list-
	echo 'urlcash.net' >> /etc/dbab/dbab.list-

	/usr/sbin/dbab-get-list
	grep googleadservices  /etc/dnsmasq.d/dbab.*

	service dnsmasq restart 

	dig @127.0.0.1 www.googleadservices.com

It should show real IP instead of `127.0.0.1`.

### dnsmasq: setting capabilities failed

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

