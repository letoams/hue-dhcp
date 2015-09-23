hue-dhcp
========

Turn on your Philips Hue lights when it is dark and your phone joins your wifi network

The goal
========

I was tired of stumbling in my dark hallway with my phone trying to
turn my lights when it was dark. So now my DHCP server tells my hue
bridge to turn on my lights if my phone joins my wifi and it is dark
outside. This method is quick enough to turn on my lights while I'm
still unlocking or opening my front door.

Requirements
============

* phue python library https://github.com/studioimaginaire/phue

* ISC dhcp server http://www.isc.org/

* geoip.prototypeapp.com (free service)

* sunrise-sunset.org (free service)

INSTALLATION
============

Copy hue-dhcp to /usr/local/sbin (or change the execute() command below)

Add the following to your dhcpd.conf:

```
on commit {
        set ClientIP = binary-to-ascii(10, 8, ".", leased-address);
        set ClientMac = binary-to-ascii(16, 8, ":", substring(hardware, 1, 6));
        log(concat("Commit: IP: ", ClientIP, " Mac: ", ClientMac));
        execute("/usr/local/sbin/hue-dhcp", "commit", ClientIP, ClientMac);
}
```

Since the phue python library stores the bridge location and password in a
file ~/.python\_hue, the dhcpd must run as a user with a home directory.
Normally it runs as user nobody which has no home directory. You can change
it to use root using the -user root -group root option. For example, on Fedora
and RHEL7/CentOS7 run:

```
cp /lib/systemd/system/dhcpd.service /etc/systemd/system/dhcpd.service
```

then edit /etc/systemd/system/dhcpd.service and replace:
```
ExecStart=/usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid
```
with:
```
ExecStart=/usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user root -group root --no-pid
```

and run:

```
systemctl daemon-reload
systemctl dhcpd restart
```

CONFIGURATION
=============

For the phue library to login to the bridge, it needs the location
and password of the Philips Hue bridge. You can either copy your
own ~/.python_hue file into /root/ or you can let root generate
one by pressing the button on the bridge and manually running
/usr/local/sbin/hue-dhcp once (as the root user)


There are three variables you can customize in the hue-dhcp script:

```
mymacs = []
# mymacs = [ "f0:d1:a9:00:00:00" , "28:e1:4c:00:00:00" ]
```

If your wifi network is protected, you can allow any device that joins
to cause the lights to go on. If you run an open wifi network (like I
do) then you will want to edit the mymacs variable in hue-dhcp to only
trigger on your phone's mac address.

```
# mylights = [ "window", "ceiling1", "ceiling2", "ceiling3" ]
mylights = []
```

When left empty, all the lights are turned on. Otherwise only the named
lights are turned on.

```
blink = False
# blink = True
```

If the lights are already on, you can blink the light in acknowledgement
when one of your allowed devices joins the network when it is dark
outside.

LICENSE
=======
The code is in the public domain. If you use this program, feel free to
send me an email at paul@nohats.ca to let me know it has been useful to you.

