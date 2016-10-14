Setting up Kali Linux 2016.2 + Whonix Gateway in VirtualBox for anonymous penetration testing
------------------------------------------------------------------------


----------


----------


----------


## Downloading and importing Whonix-Gateway into VirtualBox  ##
Kindly refer to whonix docs:
https://www.whonix.org/wiki/VirtualBox


----------
## Creating New Virtual Machine with downloaded Kali ISO image ##
In VirtualBox Manager:

 1. New => Name (Kali) => Type: Linux => Version: Debian (64)
 2. 4096 Mb
 3. Create a virtual hard drive now
 4. VDI
 5. Dynamically allocated
 6. 15-25 Gb


----------
## Changing Kali Virtual Machine settings ##
In VirtualBox Manager:
Select newly created Kali machine => Settings

 1. Network => Adapter 1 => attached to **Internal Network**
 2. Network => Adapter 1 => Name: **Whonix**
 3. System => Motherboard => **"Hardware clock in UTC"**
 4. System => Motherboard => Pointing Device => **PS/2 Mouse**
 5. System => Processor => **Enable PAE/NX**
 6. USB => **disable "Enable USB cotroller"**
 


----------
## Installing Kali ##
Power off Whonix Gateway and Start Kali Virtual Machine.

 1. Choose your Kali Linux ISO image
 2. Install
 3. English
 4. United States
 5. American English
 6. Do not configure the network at this time
 7. Hostname: kali
 8. Choose password for root
 9. Eastern
 10. Default Partition
 11. Use network mirror => No
 12. Install grub
 13. Restart


----------
## Configuring network ##
Remove network-manager:

    root@kali:~# apt-get remove network-manager
Open /interfaces:

    root@kali:~# nano /etc/network/interfaces
And add this strings:

>     auto eth0
>     iface eth0 inet static
>     address 10.152.152.12
>     netmask 255.255.192.0
>     gateway 10.152.152.10

Specify dns:

    root@kali:~# echo nameserver 10.152.152.10 > /etc/resolv.conf


----------
## Adding packages source ##
Open sources.list:

    root@kali:~# nano /etc/apt/sources.list
And add this string ( check http://docs.kali.org/general-use/kali-linux-sources-list-repositories ):

    deb http://http.kali.org/kali kali-rolling main contrib non-free


----------
## Additional security tips ##
Disable TCP_timestamps

> Disabling this will hide current uptime of your machine.

    root@kali:~# echo “net.ipv4.tcp_timestamps = 0” > /etc/sysctl.d/tcp_timestamps.conf
Change time in UTC format:

    root@kali:~# ln -fs /usr/share/zoneinfo/Etc/UTC /etc/localtime
    root@kali:~# echo UTC0 > /etc/localtime
Reboot machine:

    root@kali:~# reboot


----------
Switch to VirtualBox Manager and start Whonix Gateway.


----------
Upgrade and reboot your Kali machine:

    root@kali:~# apt-get update && apt-get dist-upgrade
    root@kali:~# reboot


----------
## Fixing screen resolution in Kali machine(optional) ##

    root@kali:~# apt-get install -y linux-headers-$(uname -r)
In the VirtualBox window with Kali select Devices => Insert Guest Additions CD image.

    root@kali:~# reboot
    root@kali:~# cd /media/cdrom
    root@kali:~# sh VBoxLinuxAdditions.run
    root@kali:~# reboot



----------
## Downloading and configuring Tor Browser ##
Download tor browser (https://www.torproject.org/)
Enable tor for root user:

    root@kali:~# nano ./tor-browser_en-US/Browser/start-tor-browser
Find and comment this strings:

>     #if [ "`id -u`" -eq 0 ]; then
>     #       complain "The Tor Browser Bundle should not be run as root.  Exiting."
>     #       exit 1
>     #fi

Change owner:

    root@kali:~# chown -R root:root ./tor-browser_en-US/


----------
## AVOIDING TOR OVER TOR ##
Now we have tor-browser through transparent proxy over whonix-gateway.
Tor over tor, that is not good.

https://trac.torproject.org/projects/tor/wiki/doc/TorifyHOWTO#ToroverTor
> When using a transparent proxy, it is possible to start a Tor session from the client as well as from the transparent proxy, creating a "Tor over Tor" scenario. Doing so produces undefined and potentially unsafe behavior. In theory, however, you can get six hops instead of three, but it is not guaranteed that you'll get three different hops - you could end up with the same hops, maybe in reverse or mixed order. It is not clear if this is safe. It has never been discussed. 
> You can ​choose an entry/exit point, but you get the best security that Tor can provide when you leave the route selection to Tor; overriding the entry / exit nodes can mess up your anonymity in ways we don't understand. Therefore Tor over Tor usage is highly discouraged.

To avoid Tor over Tor install rinetd:

    root@kali:~# apt-get install rinetd
Open rinetd.conf :

    root@kali:~# nano /etc/rinetd.conf
And add:

>     ## SocksPorts
>     ## Tor's default port
>     127.0.0.1        9050      10.152.152.10    9050
>     ## Tor Browser Bundle's default port
>     127.0.0.1        9150      10.152.152.10    9150
>     ## TorChat's default port
>     127.0.0.1        11109     10.152.152.10    9119
>     ## Tor Messenger's default port
>     127.0.0.1        9152      10.152.152.10    9152
>      
>     ## ControlPorts
>     ## Control Port Filter Proxy is running on Gateway's Port 9052
>     ## Tor's default port
>     127.0.0.1        9051      10.152.152.10    9052
>     ## Tor Browser Bundle's default port
>     127.0.0.1        9151      10.152.152.10    9052
>     ## Tor Messenger's default port
>     127.0.0.1        9153      10.152.152.10    9052

Then:

    root@kali:~# nano ~/.profile
Add:

>     export TOR_SKIP_LAUNCH=1
>     export TOR_SKIP_CONTROLPORTTEST=1
>     export TOR_NO_DISPLAY_NETWORK_SETTINGS=1
>     export TOR_CONTROL_HOST="127.0.0.1"
>     export TOR_CONTROL_PORT="9151"
>     export TOR_CONTROL_PASSWD='"password"'

Done. No more Tor over Tor


----------
## Disabling transparent proxy ##
Switch to Whonix Gateway and open terminal:

    user@host:/$ sudo nano /etc/whonix_firewall.d/30_default.conf
Find this strings:

>     WORKSTATION_TRANSPARENT_TCP=1
>     WORKSTATION_TRANSPARENT_DNS=1

and change them to:

>     WORKSTATION_TRANSPARENT_TCP=0
>     WORKSTATION_TRANSPARENT_DNS=0


----------
## Stream Isolation ##
Install torsocks:

    root@kali:~# apt-get install torsocks
Open https://github.com/Whonix/uwt/blob/master/usr/bin/uwt and copy this to:

    root@kali:/# nano /usr/bin/uwt
Save and allow execution:

    root@kali:~# chmod +x /usr/bin/uwt
Execute:

    root@kali:~# uwt
Output should be like this:

>     Usage: uwt [-h] [-v] -i ip -p port <command> [<options>...]
>     Example: uwt -i 127.0.0.1 -p 9050 wget https://check.torproject.org
>     sudo uwt -i 10.152.152.10 -p 9104 /usr/bin/apt-get --yes dist-upgrade

Example:

    root@kali:~# alias apt-get='uwt -i 10.152.152.10 -p 9104 apt-get'
    


----------
Now your machine is ready.
Reboot it and work.
