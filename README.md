# Setting up Ubuntu Host as a DnsServer
<p>
1.install the bind9 or bind package.Use link to download the binaries and install it 
https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjLz6jb2pX1AhVFT2wGHVZEBPYQFnoECAIQAQ&url=https%3A%2F%2Fpackages.ubuntu.com%2Fbind9&usg=AOvVaw3jYBqSeOvwBADFW9ZzlaDK or use can use the package manager based on your distro to install it
</p>
<h3> what is DnsServer? </h3> /etc/host
<p>
  In our day to day life we are using the dns and dnserver all the time.Our devices(Servers,Computes,Cellphones) are talking to each other using their ipaddress.if you want to get any website to load in tour browser you need its ipaddress.But as a human it is impossible to remember everything.Or we would use some old techniques like noting down the name and mobile number of the person in the phonebook :)
  So for the easy purpose each and every website name and their ipaddress will be hosted in Dnserver.We can think that as a common phonebook for everyone .so they can use it .when we type something in the browser first it will check in the /etc/hosts file in the ubuntu to see whether the ipaddress of that domain exist or not,it wont be until you manually editting the file.And after that it will see the /etc/resolv.conf file for the destination dnsserver ipaddress.
