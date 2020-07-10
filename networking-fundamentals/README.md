# Networking Fundamentals

Base-line Foundations of networking fundaments to become a great solutions architect.

## Seven layer OSI networking Model

First published in 1984!

Each layer uses the layer below and adds additional capabilities. Data between two devides moves down the stack at the each device by wrapped at each layer and then transmitted, then wrapping stripped at each layer, called encapsulation.

__All people seem to need data processing!__

7. __Application__
    - __HTTP, SSH, FTP__ are added at L7

6. __Presentation__
    - TLS encryption
    - Adds __data converion, encryption, compression and standards__ L7 can use
    - __HTTP L7 running oer TLS(L6) is HTTPS__

5. __Session__ (stateful tcp/80)
    - Adds concept of sessions, so that __request and reply communication streams are viewed as a single 'session'__ of communication between client and server

4. __Transport__ (stateless tcp/80, tcp/22)
    - Reliability
    - Adds TCP/UDP
    - __TCP__ is designed for __reliable transport__, __UDP is aimed at speed__.
    - TCP used __segments__ to ensure data is received in the __correct order__, adds __error checking__ and __ports__ allowing different streams of communications to the same host(e.g. tcp/22 and tcp/80)

3. __Network__
    - End-to-end communications
    - allows for __unique devide to device communication__ over interconnected networks
    - L3 packets travel through different L2 frames as they pass through different networks

2. __Data link__
    - base level networking
    - __Adds MAC addresses__ which can be used to named communication between two devices on a local network
    - Adds controls over the media, avoiding cross-talk and allows for backoff and retransmission
    - L2 runs on top of L1, L2 utilizes L1 to broadcast and listen.

1. __Physical__
    - base level networking
    - layer 1 networks use a __shared hardware__


## IP addressing basics

IPv4 - __32-bit(4 byte) binary values__ - represented in dotted-decimal noation

IP addresses are how two devices can communicate at layer-4 and above of the OSI 7-layer model.

192.168.10.5

Each is 8 bit - 11000000 10101000 00001010 00000101

__8 bit = 1 byte__

Two parts: Network and Host
    - prefix /24 ==> available IPS are 8-bit - 254 available IPs
    - prefix /16 ==> available IPS are 16-bit - 256*256(65534) available IPs

- 0.0.0.0/0 - Represents all IP addresses (up-to 4 Billions worldwide)
- 255.255.255.255 - to broadcast to all IP addresses (Filtered and not passed between networks)
- 127.0.0.1 - Localhost or loopback (device refers itself)
- 169.254.0.1 to 169.254.255.254 - A range of IP addresses which can auto configure with if its using DHCP.

Historically IP addresses classes:
    - Class A(/8) -- 1.0.0.0 to 126.255.255.255 -- 126 Networks -- 16,777,214 Nodes in each
    - Class B(/16) -- 128.0.0.0 to 191.255.255.255  -- 16,382 Networks -- 65,534
    - Class C(/24) -- 192.0.0.0 to 223.255.255.255 -- 2,097,150 Networks -- 254 Nodes in each

It failed, so welcome __CIDR__(Classless Inter-Domain Routing)!

__Private IP classe ranges__
    - __10.0.0.0__ to __10.255.255.255__ private networking from Class A
    - __172.16.0.0__ to __172.31.255.255__ private from Class B(16 Networks)
    - __192.168.0.0__ to __192.168.255.255__ private from class C(256 networks)

Thse ranges are often used on private business networks, cloud networks and home networks

__CIDR - 10.0.0.0/16 network/prefix__
    - is used for IPv4 IP networking rather than the class system
    - it allows more effective allocation and sub networking
    - __Network part__ remains the same and __node part__ is all yours from all 0's to all 1's
    - 10.0.0.0/16 ==> 10.0.0.0 to 10.0.255.255
    - 10.0.0.0/24 ==> 10.0.0.0 to 10.0.0.255


## Subnetting

Subnetting is a process of breaking a network down into smaller sub-networks. E.G. VPC subnets.

Public subnet - 10.0.0.0/16 (65,536 addresses)
Two subnets:  10.0.0.0/17 and 10.0.128.0/17
Four subnets: 10.0.0.0/18 and 10.0.64.0/18 and 10.0.128.0/18 and 10.0.192.0/18

    - With a certain size of VPC, increasing the prefix creates 2 smaller sized networks.
    - Increasing again, creates 4 even smaller networks..so on


## IP routing

local device-to-device L1 and L2. L3 takes care of the Origin and Destination. At the end of the day, it's just good old fashioned L2.
1. Local
2. Remote
3. Unknown

- Internet uses a routing protocol called Border Gateway Protocol(BGP)
- This protocoal exchanges routes

This is how internet works, Unchanged packets being passed around from router to router, each time using a new L2 connection.

Its connects multiple networks and forwards packets destined either for its own network or remote networks.

## Firewalls

A firewall is a device which historically sits at the border between different networks, and monitors traffic between them, It can read packat data and either allowing or dening traffic based on that data.

What data a firewall can read and act on depends on the OSI lar the firewall operates at:
    - layer 3(__Network__) - Source/Destination IP addresses Ranges, __Network LB__
    - Layer 4(__Transport__) - Protocol (TCP/UDP) & Port numbers, __NACLs__
    - Layer 5(__Session__) - As layer 4, but understand response traffic, __Security Groups in AWS__
    - Layer 6(__Application__) - Application specifics. Html paths, images, __Application LB__

__ARP__ Translates an __IP into a MAC address__

## Proxy servers

A Proxy Server is a type of Gateway which Sits at the boundary of private network and connects with Public Internet.

Proxy Serers can provide
    - Filter safety: child safety, malware, removing adult content
    - Web cache, speeding delivery of data
    - Pass trafic or not based on __things that network layer application can't__. Like: __Security privilege__ or __DNS rather than IP__
    - can perform custom validations, checks, application data, authorization
    - Outbound filtering based on application values
    - Can be installed on an EC2
    - Proxy servers can filter based on internal security privileges
    - They can offer network filtering, But Other AWS filtering products already does this
    - Proxy servers do not handle unsolicited inbound traffic as they do not act as a firewall.
