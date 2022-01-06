# Setting up Ubuntu Host as a DnsServer



 * Install the bind9 or bind package.Use link to download the binaries and install it 
https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjLz6jb2pX1AhVFT2wGHVZEBPYQFnoECAIQAQ&url=https%3A%2F%2Fpackages.ubuntu.com%2Fbind9&usg=AOvVaw3jYBqSeOvwBADFW9ZzlaDK or use can use the package manager based on your distro to install it

<h3> what is DnsServer? </h3> 

 * In our day to day life we are using the dns and dnserver all the time.Our devices(Servers,Computes,Cellphones) are talking to each other using their ipaddress.if you want to get any website to load in tour browser you need its ipaddress.But as a human it is impossible to remember everything.Or we would use some old techniques like noting down the name and mobile number of the person in the phonebook :)
  
 * So for the easy purpose each and every website name and their ipaddress will be hosted in Dnserver.We can think that as a common phonebook for everyone .so they can use it .when we type something in the browser first it will check in the /etc/hosts file in the ubuntu to see whether the ipaddress of that domain exist or not,it wont be until you manually editting the file.And after that it will see the /etc/resolv.conf file for the destination dnsserver ipaddress.

  <h2> We will set up caching nameserver and as well as primary name server</h2>
    
  <h3> Steps </h3>

  1.Backup /etc/hosts, /etc/resolv.conf, /etc/named.conf, /etc/sysconfig/iptables
  - iptables file is not require if you don't use any kind of firewall in your host

  2.go to /etc/resolv.Add the below line 
  - nameserver 192.168.0.0(your ipaddress)

  3.To see your ipaddress use ifconfig in Bashshell
  - nameserver by default . if you are using anykind of wireless connection for accessing the internet .Then that device ipaddress will be presented there by 
default.This will be done by the NetworkManager Daemon in ubuntu

  4.Now your host will be acting as a DnsServer for you.If you try to access the internet . You will get some connection error,because our host machine donot know any ipaddress of Authoritive Server ipaddress

   <h3>Setting a cache nameserver</h3>
   
   1.It is not a authoritative server for any domain,it is just a cache server for your operating system.So it don't need do the dns resolution for same website each and everytime,but there will be timelimit for each cache ipaddress in caching server
    
   2.By default BIND will refer to the root server for the dns queries ,we can also give the another destination type which is FORWARDER,where the public dnsserver will be presented such as google dnsserver or cloudflare dnsserver,both of these are the mainly used public dnsserver,and those server would have cache your required or requested domain.Or it will query the root server for the ipaddress.
    
   *Add the below lines in the /etc/named.conf
   ***
    
    
  ```
                        options {
                                listen-on port 53 { 127.0.0.1; your_private_ipadrress; };
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
 ```

  - Now restart the named Daemon
      systemctl enable named
      systemctl start named
      
  - You can check whether caching nameserver working or not by using the dig comment in the terminal
  
  ***
  
  
 <h4> Primary Namerserver </h4>

* Setting a Primary Namerserver is simple,but the files name will be differ for each linux distro.Make sure to google it.
  
* The two new files you'll create are the forward and reverse zone files, which you'll place in the /var/named directory. This location is specified by the "directory" directive in the named.conf configuration file
  
  <h5>** Create a Forward Zone File** </h5>
  
* The two new files you'll create are the forward and reverse zone files, which you'll place in the /var/named directory. This location is specified by the "directory" directive in the named.conf configuration file
    
 *  Create a basic forward zone file, /var/named/example.com.zone, and add the following lines to it. Your zone file should look like the sample zone file in Listing 3, below, when you're finished.

```
   
  - ; Authoritative data for example.com zone
     ;
     $TTL 1D
                    @   IN SOA  epc.example.com   root.epc.example.com. (
                                                           2017031301      ; serial
                                                           1D              ; refresh
                                                           1H              ; retry
                                                           1W              ; expire
                                                           3H )            ; minimum

                    $ORIGIN         example.com.
                    example.com.            IN      NS      yourdomainname
                    epc                     IN      A       127.0.0.1
                    server                  IN      A       your_webserver_ip
                    www                     IN      CNAME   server
                    mail                    IN      CNAME   server
                    test1                   IN      A       192.168.25.21
                    t1                      IN      CNAME   test1
                    
                    ; Mail server MX record
                    example.com.            IN      MX      10      mail.example.com.
```
    
* You should always provided a Fully Qualified Domain Name (FQDN).More about url https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwifgeTTppj1AhW5TGwGHcClBt0QFnoECAsQAw&url=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FLearn%2FCommon_questions%2FWhat_is_a_URL&usg=AOvVaw2JONnAJ-2axTgagyOUfhK4
    
* Add the forward zone files to named.conf

* Before your DNS server will work, however, you need to create an entry in /etc/named.conf that will point to your new zone file. Add the following lines below the entry for the top-level hints zone but before the "include" lines.
     
     
                        zone "example.com" IN {
                                type master;
                                file "example.com.zone";
                        };
                      
 * Now restart named to make these changes take effect. Test your name server by using the dig and nsloookup commands to obtain the IP Addresses for the hosts you have configured in the forward zone file. Note that the host does not have to exist on the network for the dig and nslookup commands to return an IP Address.
 * Use dig command to check
 
 <h3> Using the root name servers </h3>
 * Notice that the root name servers are given as the authoritative servers for the Amazon.com lookup. But remember you're using the Google public name servers as forwarders. Now comment out the forwarders line in named.conf and restart named. Run the above commands again to compare the results that are returned.
  ```
                             

                          ; <<>> DiG 9.10.4-P6-RedHat-9.10.4-4.P6.fc25 <<>> www.amazon.com
                          ;; global options: +cmd
                          ;; Got answer:
                          ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 65004
                          ;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 4, ADDITIONAL: 1

                          ;; OPT PSEUDOSECTION:
                          ; EDNS: version: 0, flags:; udp: 4096
                          ;; QUESTION SECTION:
                          ;www.amazon.com.                        IN      A

                          ;; ANSWER SECTION:
                          www.amazon.com.         1800    IN      CNAME   www.cdn.amazon.com.
                          www.cdn.amazon.com.     300     IN      CNAME   d3ag4hukkh62yn.cloudfront.net.
                          d3ag4hukkh62yn.cloudfront.net. 60 IN    A       52.85.147.120
                          d3ag4hukkh62yn.cloudfront.net. 60 IN    A       52.85.147.50
                          d3ag4hukkh62yn.cloudfront.net. 60 IN    A       52.85.147.92
                          d3ag4hukkh62yn.cloudfront.net. 60 IN    A       52.85.147.109

                          ;; AUTHORITY SECTION:
                          d3ag4hukkh62yn.cloudfront.net. 1831 IN  NS      ns-1144.awsdns-15.org.
                          d3ag4hukkh62yn.cloudfront.net. 1831 IN  NS      ns-130.awsdns-16.com.
                          d3ag4hukkh62yn.cloudfront.net. 1831 IN  NS      ns-2021.awsdns-60.co.uk.
                          d3ag4hukkh62yn.cloudfront.net. 1831 IN  NS      ns-824.awsdns-39.net.

                          ;; Query time: 3857 msec
                          ;; SERVER: 192.168.0.203#53(192.168.0.203)
                          ;; WHEN: Mon Mar 13 09:18:30 EDT 2017
                          ;; MSG SIZE  rcvd: 306
                          ''
  ```
    ***
<h3> Creating the reverse zone file </h3>
* A reverse zone for your domain will provide the ability to do reverse lookups. Many organizations do not do these internally, but reverse lookups can be helpful in doing problem determination. Many spam fighting configurations, such as SpamAssassin, look for reverse lookups to verify valid email servers.
* Create the reverse zone file, /var/named/example.com.rev

  ***
 
  ```
      ; Authoritative data for example.com  reverse zone
      ;
      $TTL 1D
      @   IN SOA  test1.example.com  example.com. (
                                              2017031501      ; serial
                                              1D              ; refresh
                                              1H              ; retry
                                              1W              ; expire
                                              3H )            ; minimum

      @       IN      NS      epc.example.com.
      example.com.    IN      NS      epc.example.com.
      1               IN      PTR     mail.example.com.
      1               IN      PTR     server.example.com.
      21              IN      PTR     test1.example.com.
      22              IN      PTR     test2.example.com.
      23              IN      PTR     test3.example.com.
      24              IN      PTR     test4.example.com.
  ```

    ***

* Add the reverse zone to named.conf:

    
  ``` 
        zone    "25.168.192.in-addr.arpa" IN {
       type master;
       file "example.com.rev";
        };
  ```
 
    ***
 
 *  systemctl reload named
 *  And use the dig command to see the result
