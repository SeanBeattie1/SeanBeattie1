# Scanning

## Discovering hosts from the outside

This is going to be a **brief section** about how to find **IPs responding** from the **Internet**.\
In this situation you have some **scope of IPs** (maybe even several **ranges**) and you just to find **which IPs are responding**.

### ICMP

This is the **easiest** and **fastest** way to discover if a host is up or not.\
You could try to send some **ICMP** packets and **expect responses**. The easiest way is just sending an **echo request** and expect from the response. You can do that using a simple `ping`or using `fping`for **ranges**.\
You could also use **nmap** to send other types of ICMP packets (this will avoid filters to common ICMP echo request-response).

```bash
ping -c 1 199.66.11.4    # 1 echo request to a host
fping -g 199.66.11.0/24  # Send echo requests to ranges
nmap -PEPM -sP -n 199.66.11.0/24 #Send echo, timestamp requests and subnet mask requests
```

### TCP Port Discovery

It's very common to find that all kind of ICMP packets are being filtered. Then, all you can do to check if a host is up is **try to find open ports**. Each host has **65535 ports**, so, if you have a "big" scope you **cannot** test if **each port** of each host is open or not, that will take too much time.\
Then, what you need is a **fast port scanner** ([masscan](https://github.com/robertdavidgraham/masscan)) and a list of the **ports more used:**

```bash
#Using masscan to scan top20ports of nmap in a /24 range (less than 5min)
masscan -p20,21-23,25,53,80,110,111,135,139,143,443,445,993,995,1723,3306,3389,5900,8080 199.66.11.0/24
```

You could also perform this step with `nmap`, but it slower and somewhat `nmap`has problems identifying hosts up.

### HTTP Port Discovery

This is just a TCP port discovery useful when you want to **focus on discovering HTTP** **services**:

```bash
masscan -p80,443,8000-8100,8443 199.66.11.0/24
```

### UDP Port Discovery

You could also try to check for some **UDP port open** to decide if you should **pay more attention** to a **host.** As UDP services usually **don't respond** with **any data** to a regular empty UDP probe packet it is difficult to say if a port is being filtered or open. The easiest way to decide this is to send a packet related to the running service, and as you don't know which service is running, you should try the most probable based on the port number:

```bash
nmap -sU -sV --version-intensity 0 -F -n 199.66.11.53/24
# The -sV will make nmap test each possible known UDP service packet
# The "--version-intensity 0" will make nmap only test the most probable
```

The nmap line proposed before will test the **top 1000 UDP ports** in every host inside the **/24** range but even only this will take **>20min**. If need **fastest results** you can use [**udp-proto-scanner**](https://github.com/portcullislabs/udp-proto-scanner): `./udp-proto-scanner.pl 199.66.11.53/24` This will send these **UDP probes** to their **expected port** (for a /24 range this will just take 1 min): _DNSStatusRequest, DNSVersionBindReq, NBTStat, NTPRequest, RPCCheck, SNMPv3GetRequest, chargen, citrix, daytime, db2, echo, gtpv1, ike,ms-sql, ms-sql-slam, netop, ntp, rpc, snmp-public, systat, tftp, time, xdmcp._

### SCTP Port Discovery

```bash
#Probably useless, but it's pretty fast, why not trying?
nmap -T4 -sY -n --open -Pn <IP/range>
```

## Pentesting Wifi

Here you can find a nice guide of all the well known Wifi attacks at the time of the writing:

{% embed url="https://book.hacktricks.xyz/generic-methodologies-and-resources/pentesting-wifi" %}

## Discovering hosts from the inside

If you are inside the network one of the first things you will want to do is to **discover other hosts**. Depending on **how much noise** you can/want to do, different actions could be performed:

### Passive

You can use these tools to passively discover hosts inside a connected network:

```bash
netdiscover -p
p0f -i eth0 -p -o /tmp/p0f.log
# Bettercap
net.recon on/off #Read local ARP cache periodically
net.show
set net.show.meta true #more info
```

### Active

Note that the techniques commented in [_**Discovering hosts from the outside**_](./#discovering-hosts-from-the-outside) (_TCP/HTTP/UDP/SCTP Port Discovery_) can be also **applied here**.\
But, as you are in the **same network** as the other hosts, you can do **more things**:

```bash
#ARP discovery
nmap -sn <Network> #ARP Requests (Discover IPs)
netdiscover -r <Network> #ARP requests (Discover IPs)

#NBT discovery
nbtscan -r 192.168.0.1/24 #Search in Domain

# Bettercap
net.probe on/off #Discover hosts on current subnet by probing with ARP, mDNS, NBNS, UPNP, and/or WSD
set net.probe.mdns true/false #Enable mDNS discovery probes (default=true)
set net.probe.nbns true/false #Enable NetBIOS name service discovery probes (default=true)
set net.probe.upnp true/false #Enable UPNP discovery probes (default=true)
set net.probe.wsd true/false #Enable WSD discovery probes (default=true)
set net.probe.throttle 10 #10ms between probes sent (default=10)

#IPv6
alive6 <IFACE> # Send a pingv6 to multicast.
```

### Active ICMP

Note that the techniques commented in _Discovering hosts from the outside_ ([_**ICMP**_](./#icmp)) can be also **applied here**.\
But, as you are in the **same network** as the other hosts, you can do **more things**:

* If you **ping** a **subnet broadcast address** the ping should be arrive to **each host** and they could **respond** to **you**: `ping -b 10.10.5.255`
* Pinging the **network broadcast address** you could even find hosts inside **other subnets**: `ping -b 255.255.255.255`
* Use the `-PEPM` flag of `nmap`to perform host discovery sending **ICMPv4 echo**, **timestamp**, and **subnet mask requests:** `nmap -PEPM -sP –vvv -n 10.12.5.0/24`

### **Wake On Lan**

Wake On Lan is used to **turn on** computers through a **network message**. The magic packet used to turn on the computer is only a packet where a **MAC Dst** is provided and then it is **repeated 16 times** inside the same paket.\
Then this kind of packets are usually sent in an **ethernet 0x0842** or in a **UDP packet to port 9**.\
If **no \[MAC]** is provided, the packet is sent to **broadcast ethernet** (and the broadcast MAC will be the one being repeated).

```bash
# Bettercap (if no [MAC] is specificed ff:ff:ff:ff:ff:ff will be used/entire broadcast domain)
wol.eth [MAC] #Send a WOL as a raw ethernet packet of type 0x0847
wol.udp [MAC] #Send a WOL as an IPv4 broadcast packet to UDP port 9
```

## Scanning Hosts

Once you have discovered all the IPs (external or internal) you want to scan in depth, different actions can be performed.

### TCP

* **Open** port: _SYN --> SYN/ACK --> RST_
* **Closed** port: _SYN --> RST/ACK_
* **Filtered** port: _SYN --> \[NO RESPONSE]_
* **Filtered** port: _SYN --> ICMP message_

```bash
# Full TCP port scan
sudo nmap -Pn -p- -oN alltcp_ports.txt $ip

# Full TCP port scan (safe scripts + version detection)
sudo nmap -Pn -sC -sV -p- -oN alltcp.txt $ip

# Top 20 UDP port scan
sudo nmap -Pn -sU -sV -sC --top-ports=20 -oN top_20_udp_nmap.txt $ip
```

```bash
# Top 20 TCP port scan
proxychains nmap -Pn -sT --top-ports=20 --open -oN top_20_tcp_nmap.txt $ip

# Top 1000 TCP port scan
proxychains nmap -Pn -sT --top-ports=1000 --open -oN top_1000_tcp_nmap.txt $ip

# Scan TCP ports (safe scripts + version detection)
proxychains nmap -Pn -sT -sC -sV -p 21,22,80 -oN tcp_nmap_sC_sV.txt $ip
```

```bash
# Nmap fast scan for the most 1000tcp ports used
nmap -sV -sC -O -T4 -n -Pn -oA fastscan <IP> 
# Nmap fast scan for all the ports
nmap -sV -sC -O -T4 -n -Pn -p- -oA fullfastscan <IP> 
# Nmap fast scan for all the ports slower to avoid failures due to -T4
nmap -sV -sC -O -p- -n -Pn -oA fullscan <IP>

#Bettercap Scan
syn.scan 192.168.1.0/24 1 10000 #Ports 1-10000
```

### UDP

There are 2 options to scan an UDP port:

* Send a **UDP packet** and check for the response _**ICMP unreachable**_ if the port is **closed** (in several cases ICMP will be **filtered** so you won't receive any information inf the port is close or open).
* Send a **formatted datagrams** to elicit a response from a **service** (e.g., DNS, DHCP, TFTP, and others, as listed in _nmap-payloads_). If you receive a **response**, then, the port is **open**.

**Nmap** will **mix both** options using "-sV" (UDP scans are very slow), but notice that UDP scans are slower than TCP scans:

```bash
# Check if any of the most common udp services is running
udp-proto-scanner.pl <IP> 
# Nmap fast check if any of the 100 most common UDP services is running
nmap -sU -sV --version-intensity 0 -n -F -T4 <IP>
# Nmap check if any of the 100 most common UDP services is running and launch defaults scripts
nmap -sU -sV -sC -n -F -T4 <IP> 
# Nmap "fast" top 1000 UDP ports
nmap -sU -sV --version-intensity 0 -n -T4 <IP>
# You could use nmap to test all the UDP ports, but that will take a lot of time
```

### SCTP Scan

SCTP sits alongside TCP and UDP. Intended to provide **transport** of **telephony** data over **IP**, the protocol duplicates many of the reliability features of Signaling System 7 (SS7), and underpins a larger protocol family known as SIGTRAN. SCTP is supported by operating systems including IBM AIX, Oracle Solaris, HP-UX, Linux, Cisco IOS, and VxWorks.

Two different scans for SCTP are offered by nmap: _-sY_ and _-sZ_

```bash
# Nmap fast SCTP scan
nmap -T4 -sY -n -oA SCTFastScan <IP>
# Nmap all SCTP scan
nmap -T4 -p- -sY -sV -sC -F -n -oA SCTAllScan <IP>
```

### Network Scan with nc and ping

Sometimes we want to perform network scan without any tools like nmap. So we can use the commands `ping` and `nc` to check if a host is up and which port is open. To check if hosts are up on a /24 range

```bash
for i in `seq 1 255`; do ping -c 1 -w 1 192.168.1.$i > /dev/null 2>&1; if [ $? -eq 0 ]; then echo "192.168.1.$i is UP"; fi ; done
```

To check which ports are open on a specific host

```bash
for i in {21,22,80,139,443,445,3306,3389,8080,8443}; do nc -z -w 1 192.168.1.18 $i > /dev/null 2>&1; if [ $? -eq 0 ]; then echo "192.168.1.18 has port $i open"; fi ; done
```

Both at the same time on a /24 range

```bash
for i in `seq 1 255`; do ping -c 1 -w 1 192.168.1.$i > /dev/null 2>&1; if [ $? -eq 0 ]; then echo "192.168.1.$i is UP:"; for j in {21,22,80,139,443,445,3306,3389,8080,8443}; do nc -z -w 1 192.168.1.$i $j > /dev/null 2>&1; if [ $? -eq 0 ]; then echo "\t192.168.1.$i has port $j open"; fi ; done ; fi ; done
```

Not in one-liner version:

```bash
for i in `seq 1 255`; 
do 
    ping -c 1 -w 1 192.168.1.$i > /dev/null 2>&1; 
    if [ $? -eq 0 ]; 
    then 
        echo "192.168.1.$i is UP:"; 
        for j in {21,22,80,139,443,445,3306,3389,8080,8443}; 
        do 
            nc -z -w 1 192.168.1.$i $j > /dev/null 2>&1; 
            if [ $? -eq 0 ]; 
            then 
                echo "\t192.168.1.$i has port $j open"; 
            fi ; 
        done ; 
    fi ; 
done
```

### IDS and IPS evasion

{% embed url="https://book.hacktricks.xyz/generic-methodologies-and-resources/pentesting-network/ids-evasion" %}

### Rustscan

[rustscan](https://github.com/RustScan/RustScan#-usage) - Scans all 65k ports in 3 seconds and pipe them to NMAP

```
rustscan -a 127.0.0.1 -- -A -sC 
#it's like running nmap -Pn -vvv -p $PORTS -A -sC 127.0.0.1
```

### Revealing Internal IP Addresses

Misconfigured routers, firewalls, and network devices sometimes **respond** to network probes **using nonpublic source addresses**. You can use _tcpdump_ used to **identify packets** received from **private addresses** during testing. In this case, the _eth2_ interface in Kali Linux is **addressable** from the **public Internet** (If you are **behind** a **NAT** of a **Firewall** this kind of packets are probably going to be **filtered**).

```bash
tcpdump –nt -i eth2 src net 10 or 172.16/12 or 192.168/16
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth2, link-type EN10MB (Ethernet), capture size 65535 bytes
IP 10.10.0.1 > 185.22.224.18: ICMP echo reply, id 25804, seq 1582, length 64
IP 10.10.0.2 > 185.22.224.18: ICMP echo reply, id 25804, seq 1586, length 64
```

## Sniffing

Sniffing you can learn details of IP ranges, subnet sizes, MAC addresses, and hostnames by reviewing captured frames and packets. If the network is misconfigured or switching fabric under stress, attackers can capture sensitive material via passive network sniffing.

If a switched Ethernet network is configured properly, you will only see broadcast frames and material destined for your MAC address.

### TCPDump

```bash
sudo tcpdump -i <INTERFACE> udp port 53 #Listen to DNS request to discover what is searching the host
tcpdump -i <IFACE> icmp #Listen to icmp packets
sudo bash -c "sudo nohup tcpdump -i eth0 -G 300 -w \"/tmp/dump-%m-%d-%H-%M-%S-%s.pcap\" -W 50 'tcp and (port 80 or port 443)' &"
```

One can, also, capture packets from a remote machine over an SSH session with Wireshark as the GUI in realtime.

```
ssh user@<TARGET IP> tcpdump -i ens160 -U -s0 -w - | sudo wireshark -k -i -
ssh <USERNAME>@<TARGET IP> tcpdump -i <INTERFACE> -U -s0 -w - 'port not 22' | sudo wireshark -k -i - # Exclude SSH traffic
```

### Bettercap

```bash
net.sniff on
net.sniff stats
set net.sniff.output sniffed.pcap #Write captured packets to file
set net.sniff.local  #If true it will consider packets from/to this computer, otherwise it will skip them (default=false)
set net.sniff.filter #BPF filter for the sniffer (default=not arp)
set net.sniff.regexp #If set only packets matching this regex will be considered
```
