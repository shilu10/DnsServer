# Setting up Ubuntu Host as a DnsServer


<p>
1.install the bind9 or bind package.Use link to download the binaries and install it 
https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjLz6jb2pX1AhVFT2wGHVZEBPYQFnoECAIQAQ&url=https%3A%2F%2Fpackages.ubuntu.com%2Fbind9&usg=AOvVaw3jYBqSeOvwBADFW9ZzlaDK or use can use the package manager based on your distro to install it
</p>
<h3> what is DnsServer? </h3> 
<p>
  In our day to day life we are using the dns and dnserver all the time.Our devices(Servers,Computes,Cellphones) are talking to each other using their ipaddress.if you want to get any website to load in tour browser you need its ipaddress.But as a human it is impossible to remember everything.Or we would use some old techniques like noting down the name and mobile number of the person in the phonebook :)
  So for the easy purpose each and every website name and their ipaddress will be hosted in Dnserver.We can think that as a common phonebook for everyone .so they can use it .when we type something in the browser first it will check in the /etc/hosts file in the ubuntu to see whether the ipaddress of that domain exist or not,it wont be until you manually editting the file.And after that it will see the /etc/resolv.conf file for the destination dnsserver ipaddress.

 <h2> We will set up caching nameserver and as well as primary name server</h2>
    
<h3> Steps </h3>

1.Backup /etc/hosts, /etc/resolv.conf, /etc/named.conf, /etc/sysconfig/iptables
iptables file is not require if you don't use any kind of firewall in your host

2.go to /etc/resolv.Add the below line 
-nameserver 192.168.0.0(your ipaddress)

-To see your ipaddress use ifconfig in Bashshell
-nameserver by default . if you are using anykind of wireless connection for accessing the internet .Then that device ipaddress will be presented there by 
default.This will be done by the NetworkManager Daemon in ubuntu

-Now your host will be acting as a DnsServer for you.If you try to access the internet . You will get some connection error,because our host machine donot know any ipaddress of Authoritive Server ipaddress

-Setting a cache nameserver
    -It is not a authoritative server for any domain,it is just a cache server for your operating system.So it don't need do the dns resolution for same website each and everytime,but there will be timelimit for each cache ipaddress in caching server
    
    -By default BIND will refer to the root server for the dns queries ,we can also give the another destination type which is FORWARDER,where the public dnsserver will be presented such as google dnsserver or cloudflare dnsserver,both of these are the mainly used public dnsserver,and those server would have cache your required or requested domain.Or it will query the root server for the ipaddress.
    
    -Add the below lines in the /etc/named.conf
    
    
   <p>
                        // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
                        // server as a caching only name server (as a localhost DNS resolver only).
                        // See /usr/share/doc/bind*/sample/ for example named configuration files.
                        //
                        //
                        
                        options {
                                listen-on port 53 { 127.0.0.1; 192.168.0.203; };
                        //      listen-on-v6 port 53 { ::1; };
                                forwarders { 8.8.8.8; 8.8.4.4; };
                                directory       "/var/named";
                                dump-file       "/var/named/data/cache_dump.db";
                                statistics-file "/var/named/data/named_stats.txt";
                                memstatistics-file "/var/named/data/named_mem_stats.txt";
                                allow-query     { localhost; 192.168.0.0/24; };
                                recursion yes;


                                dnssec-enable yes;
                                dnssec-validation yes;
                                dnssec-lookaside auto;


                                /* Path to ISC DLV key */
                                bindkeys-file "/etc/named.iscdlv.key";


                                managed-keys-directory "/var/named/dynamic";
                        };
                        logging {
                                channel default_debug {
                                        file "data/named.run";
                                        severity dynamic;
                                };
                        };
                        zone "." IN {
                                type hint;
                                file "named.ca";
                        };
                        include "/etc/named.rfc1912.zones";
                        include "/etc/named.root.key";
  </p>

  -Now restart the named Daemon
      systemctl enable named
      systemctl start named
      
  -You can check whether caching nameserver working or not by using the dig comment in the terminal

