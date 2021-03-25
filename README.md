

## Dnsspoof attack vulnerability
Recently, an Israel Hebrew University’s cyber security research labs, JSOF, announced
“DNSpooq - Kaminsky attack is back!”1. 7 new vulnerabilities are being disclosed in common DNS software dnsmasq, reminiscent of 2008 weaknesses in Internet DNS Architecture. Three of them related to Cache Poisoning Vulnerabilities which are labeled as CVE-2020-25684, CVE-2020-25685, CVE-2020-25686. And the remained four are Buffer Overflow Vulnerabilities which are labeled as CVE-2020-25687, CVE-2020-25683 ,CVE-2020-25682 and CVE-2020-25681.
The Common Vulnerabilities and Exposures (CVE) of this analysis report are mainly focus on the Cache Poisoning Vulnerabilities part, which are labeled as CVE-2020-25684. this DNSpooqs(DNS spoofing) was listed in the common vulnerabilities and exposures list2 in 16th September, 2020.

dnsmasq limits the number of unanswered queries that are forwarded to an upstream server. According to the default setting, the maximum number of forwarded queries is 150. Therefore, if the maximum number of forwarded queries reached, the oldest queries will be dropped.
The forwarded queries are represented using the frec (forward record) structure. The forward record structure is as same as DNS header packet structure. Each frec is paired with a 16 bit transaction ID (TXID) of the forwarded query. According to the default setting, the maximum number of sockets used for forwarding is 64.
Each forward socket is bound to a random source port. In the DNS request packet, TXID and source port added 32 bit. However, since dnsmasq used same source port for multiple TXIDs. As result, the attacker only correctly guess one of 64 port, which is 1.5% successfully rate. And then correctly guess the TXID which formed by 16 bit. As result, since dnsmasq used the same port for multiple TXIDs. This increase the chance for the attacker to guess the port and IP of the DNS packet and make a fake DNS query and reply.
In the picture below showed an instance, the user want to visit google.com, the client PC send the DNS request packet to DNS Server through the internet from local dnsmasq service. When the client PC waiting the DNS Server reply, the attacker send the fake DNS server reply packet to dnsmasq service. This will lead the user to a fraud and malicious website.

![image](https://user-images.githubusercontent.com/76594282/112460303-a6cba580-8d56-11eb-9f77-ef97383db20b.png)

### Built With

This section should list any major frameworks that bult for this project:
* [Docker](https://www.docker.com/)
* [Python 3.7](https://www.python.org/)
* [Perl Source Code](https://www.perl.org/get.html) 
* [CentOS](https://www.centos.org/)
* [Wireshark](https://www.wireshark.org/)
* [Dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html)
* [Kali](https://www.kali.org/)
  
  Ettercap, Bettercap, Dnschef, metasploit, nmap ...




<!-- GETTING STARTED -->
## Getting Started

To build up the environment of the project by using Docker image. 
A prepared image was built. Image included all environment and tool. 

1) To access the docker image, it require the privileged premeission to run the system service. It need to run the bash by using /usr/sbin/init 
  docker run -d -t --name [container name] --cap-add=NET_ADMIN -h [host name] --privileged=true [the image that provided] /usr/sbin/init 
2) To start that container 
  docker start <container-name/ID>
3) To run the bash of that container
  docker exec -it <container-name/ID> bash

### Installation
install the programme:

Python 3.7
1) Download GCC compiler 
  yum install gcc openssl-devel bzip2-devel libffi-devel zlib-devel
2) Download Python 3.7
  #cd /usr/src
  #wget https://www.python.org/ftp/python/3.7.9/Python-3.7.9.tgz
  #tar xzf Python-3.7.9.tgz
3) Install Python 3.7
  #cd Python-3.7.9
  #./configure --enable-optimizations
  #make altinstall
4) Check the Pyhton Verision 
  #python3.7 -V
  
Wireshark 1.10.14
  #yum install wireshark-gnome

Dnsmasq 2.83
  #yum install dnsmasq

## Structure of the system 
![image](https://user-images.githubusercontent.com/76594282/112459341-a252bd00-8d55-11eb-9b08-80ee3489c665.png)

## Configuration for dnsmasq 

Set up DNS server via Dnsmasq
To set up dnsmasq as a DNS caching daemon on a single computer specify a listen-address directive, adding in the localhost IP address:
/etc/dnsmasq.conf
listen-address=::1,127.0.0.1

To set up the public nameserver
/etc/dnsmasq.conf
#Google's nameservers, for example
server=8.8.8.8
server=8.8.4.4

To open the port 53 for DNS server 
/etc/dnsmasq.conf
Port = 53

Uncomment expand-hosts to add the custom domain to hosts entries
/etc/dnsmasq.conf
expand-hosts

To allow all dns packet route to dnsmasq service. 
Set localhost addresses as the only nameservers in /etc/resolv.conf:
/etc/resolv.conf
nameserver ::1
nameserver 127.0.0.1
options trust-ad

Testing
#dnsmasq --test
#systemctl start dnsmasq 
#systemctl status dnsmasq 
#nslookup google.com

## Configuration for wiredshark 
To keep filtering the DNS packet
#tshark -f "src port 53" -n
#tcpdump port 53 -w capture_file
#tshark -r capture_file

To do the penetration testing of getting the transcation ID, srouce and destination ip and port, arp address(MAC). 


<!-- ROADMAP -->
## Exploit
Modern DNS protocol is using 16 bit transaction id in the Header section of a DNS message. It is easy to use brithday attack to crack the transcation id.  
![image](https://user-images.githubusercontent.com/76594282/112459006-4851f780-8d55-11eb-9ac2-13af2e0d599d.png)

Edit the Perl Source Code and edit the transaction ID, dest ip of the dns response packet.
/dns_fake_response.pl
my $sourceIP = ’10.0.0.3’; # IP address of the attacking host #(A)
my $destIP = ’10.0.0.8’; # IP address of the victim DNS server #(B)
my $destPort = 53; # usual DNS port #(C)
my $sourcePort = 5353; #(D)
#Transaction IDs to use:
my @spoofing_set = 34000..34001; # Make it to be a large and apporpriate #(E)

Use Dnschef to change the dns record 
dnschef --fakeip=192.168.0.2 --fakedomains=fake.com --interface=0.0.0.0

Use Bettercap/ettercap send the ARP message and treat the host of attacker as dns server and then change the dns records. 

## Demonstration of patch, and exploitation failure
Currently Dnsmasq already update its latest revision to 2.83 immediately fixed the loophole with latest patch after Israel Hebrew University’s cyber security research labs publish these CVEs to pbulic. It use multiple ports for multiple TXIDs and increase the security of DNSSEC validation. These CVEs currently cannot be successfully exploitated. 


<!-- CONTRIBUTING -->
## Reference 
Avinash Kak. (2021). Lecture 17: DNS and the DNS Cache Poisoning Attack. Purdue University. https://engineering.purdue.edu/kak/compsec/NewLectures/Lecture17.pdf
DNSpooq: Cache Poisoning and RCE in Popular DNS Forwarder dnsmasq. (2021). The JSOF Research Lab. https://www.jsof-tech.com/wp-content/uploads/2021/01/DNSpooq-Technical-WP.pdf
Whittle, M. (2021, January 12). Ethical Hacking (Part 9): DNS Hijacking & Credential Harvesting. Medium. https://levelup.gitconnected.com/ethical-hacking-part-9-dns-hijacking-credential-harvesting-db57302e5131
