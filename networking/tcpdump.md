### Basic Options In Tcpdump

### Tcpdump command can be used to filter all different packets.

Command | Description
---|---
tcpdump -i | any Capture from all interfaces
tcpdump -i | eth0 Capture from specific interface ( Ex Eth0)
tcpdump -i | eth0 -c 10 Capture first 10 packets and exit
tcpdump -D | Show available interfaces
tcpdump -i | eth0 -A Print in ASCII
tcpdump -i | eth0 -w tcpdump.txt To save capture to a file
tcpdump -r | tcpdump.txt Read and analyze saved capture file
tcpdump -n -i | eth0 Do not resolve host names
tcpdump -nn -i | eth0 Stop Domain name translation and lookups
tcpdump -i eth0 -c 10 -w | tcpdump.pcap tcp Capture TCP packets only
tcpdump -i eth0 port 80 | Capture traffic from a defined port only
tcpdump host 192.168.1.100 | Capture packets from specific host
tcpdump net 10.1.1.0/16 | Capture files from network subnet
tcpdump src 10.1.1.100 | Capture from a specific source address
tcpdump dst 10.1.1.100 | Capture from a specific destination address
tcpdump port 80 | Filter traffic based on a port
tcpdump portrange 21-125 | Filter based on port range
tcpdump IPV6 | Show only IPV6 packets
tcpdump -nXxei | -n unresolve dns -X -x -e -i interface


### Advanced Logical Operators in Tcpdump

Command | Description
---|---
tcpdump -n src 192.168.1.1 and dst port 21 | Combine filtering options
tcpdump dst 10.1.1.1 or !icmp | Either of the condition can match
tcpdump dst 10.1.1.1 and not icmp | Negation of the condition
tcpdump <32 | Shows packets size less than 32
tcpdump >=32 | Shows packets size greater than 32

### Tcpdump provides several options that enhance or modify its output. The following are the cheat sheet for tcpdump command.

Option | Description
---|---
-i | Listen on the specified interface.
-n | Don’t resolve hostnames. You can use -nn to don’t resolve hostnames or port names.
-t | Print human-readable timestamp on each dump line, -tttt: Give maximally human-readable timestamp output.
-X | Show the packet’s contents in both hex and ascii.
-v, -vv, -vvv | enables verbose logging/details (which among other things will give us a running total on how many packets are captured
-c N | Only get N number of packets and then stop.
-s | Define the snaplength (size) of the capture in bytes. Use -s0 to get everything, unless you are intentionally capturing less.
-S | Print absolute sequence numbers.
-q | Show less protocol information.
-w | Write the raw packets to file
-C | file_size(M) tells tcpdump to store up to x MB of packet data per file.
-G | rotate_seconds