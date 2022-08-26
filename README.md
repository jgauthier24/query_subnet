# query_subnet - a bash script

## Intro
Given a CIDR-formatted network or host address, the script queries DNS for each host address in the network range to find its  hostname, then pings it to check its status. Pinging can be disabled if all you need is to see what host addresses in the network are actually defined in DNS. DNS queries can be disabled if all you need is to see what IP host addresses are active on the network.

Output is CSV for importing into a spreadsheet.

## Background Information
A little bit of background cribbed from https://study-ccna.com/subnet-mask/ and from https://www.techtarget.com/searchnetworking/definition/CIDR

An IP address is divided into two parts: *network* and *host* parts. For example, an IP class A address consists of 8 bits identifying the network and 24 bits identifying the host. This is because the default subnet mask for a class A IP address is 8 bits long (or, written in dotted decimal notation, 255.0.0.0). What does it mean? Well, like an IP address, a subnet mask also consists of 32 bits. Computers use it to determine the network part and the host part of an address. The 1s in the subnet mask represent a network part, the 0s a host part.

Let’s say that we have the IP address of 10.0.0.1 with a subnet mask of 24 bits (255.255.255.0). First, we need to convert the IP address to binary:

```
IP address:       10.0.0.1 = 00001010.00000000.00000000.00000001
Subnet mask: 255.255.255.0 = 11111111.11111111.11111111.0000000
``` 
Computers then use the AND operation to determine the network number:

```
00001010.00000000.00000000.00000001 = 10.0.0.1
11111111.11111111.11111111.00000000 = 255.255.255.0
---------------------------------------------------
00001010.00000000.00000000.00000000 = 10.0.0.0 <- network number
```
CIDR (Classless Inter-Domain Routing) allows more granuality of subnetting than the standard Class A (255.0.0.0, 255 networks of > 16 million hosts each), Class B (255.255.0.0, 65,535 networks of 65,535 hosts each), and Class C (255.255.255.0, > 16 million networks of 255 hosts each). CIDR is based on *Variable Length Subnet Masks* (VLSM), which allows dividing an IP address space into a hierarchy of subnets of different sizes, making it possible to create subnetworks with different host counts without wasting large numbers of addresses.

CIDR addresses are made up of two sets of numbers: a prefix, which is the binary representation of the network address, similar to what would be seen in a normal IP address, and a suffix which declares the total number of bits in the entire address. For example, CIDR notation may look like `192.168.129.23/17`, with 17 being the number of bits in the address. 17 bits of subnet mask equates to 255.255.128.0 in dotted-decimal notation.

## Usage

```
query_subnet [-d|o|p] [-n domain] [-w wait_time] [-qvyh] CIDR-IP/mask
```
Optional arguments:

```
[-d|o|p]      - mutually exclusive options, where:
  -d          - ping host address only if it's in DNS,
  -o          - only query DNS, skipping pinging each host,
  -p          - ping each host address in the range, skipping DNS lookups.
[-n domain]   - overrides default of 'mitre.org' or domain found in /etc/resolv.conf.
[-w domain]   - delay in seconds between DNS lookups (defaults to $sleep_wait_time sec)
[-q]          - minimal output only 
[-v]          - print variables and other info (useful for debugging)
[-y]          - overide check against scanning more than 1022 hosts (checks for subnet mask
                length less than 23 bits)
[-h]          - print extended help info`

CIDR-IP/mask  - is a valid CIDR-style IPv4 network or host address:
                e.g. 10.10.10.0/24
```

The script automatically calculates the proper network address if you specify a host address and a netmask length, so if you don't know the network address but you do know the address of a specific host, you can specify it and an arbitrary netmask instead of having to figure out the subnet address yourself.

To ping a smaller range within a subnet, specify a larger netmask- e.g. /26 instead of /24. Be careful if pinging a range larger than a natural Class C - it can take a long time to run thru all the addresses in the range.

### Example Output
```
jgauthier@localhost:~$ ./query_subnet 192.168.129.23/25
# Given 192.168.129.23/25
# Network address: 192.168.129.0
# Netmask: 255.255.255.128
# Broadcast address: 192.168.129.127
# 126 host addresses, 192.168.129.1 to 192.168.129.126, will be queried in DNS then pinged.
IP address, DNS Name, Status
192.168.129.1, local-router, Active
192.168.129.2, No entry, Offline
192.168.129.3, my-pc, Active
192.168.129.4, my-server, Active
192.168.129.5, my-phone, Offline
192.168.129.6, No entry, Offline
192.168.129.7, No entry, Offline
...
```

IPv6 addresses not supported at this time.

To save the output, redirect to a file using 'tee' rather than just '>':
```
query_subnet 10.10.10.0/24 | tee 10.10.10.0.scan
```
That way you can see the output as well as save it to a file.
