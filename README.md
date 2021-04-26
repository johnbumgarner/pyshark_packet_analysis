<p align="center">
  <img src="https://github.com/johnbumgarner/pyshark_packet_analysis/blob/b9d356611ecbffe164cb9677347c32b00ad55009/graphic/binary_funnel.jpg" width="300" height="310"/>
</p>


# Overview Packet Analysis

<p align="justify">
This repository contains code related to the Python module <b>PyShark</b>, which is a wrapper for the <b>Wireshark</b> command-line interface (CLI) for <b>TShark</b>. The latter using the Wireshark dissectors to sniff and capture packets from a network inteface. The real power of PyShark is its capability to access to all of the packet decoders built into TShark.
</p>

<p align="justify">
PyShark can operate in either <b>LiveCapture</b> or <b>FileCapture</b> modes. Both modes have methods that can be used to parse specific packet level attributes, which includes protocols and their associated ports. 
</p>


### LiveCapture Usage examples:

<i><b>Basic Capture</b></i>

```python
capture = pyshark.LiveCapture(interface='your capture interface')
for packet in capture:
   # do something with the packet
```

<i><b>LiveCapture with packet count</b></i>

<p align="justify">
PyShark LiveCapture has a featured named <i><b>sniff_continuously</b></i> that allows you to limit the number of packets captured. 
</p>

```python
capture = pyshark.LiveCapture(interface='your capture interface')
for packet in capture.sniff_continuously(packet_count=10):
   # do something with the packet
```

<i><b>LiveCapture with timeout</b></i>

<p align="justify">
PyShark LiveCapture also has a featured named <i><b>sniff</b></i>< that allows you to set a capture timeout period. 
</p>

```python
capture = pyshark.LiveCapture(interface='your capture interface')
capture.sniff(timeout=10)
packets = [pkt for pkt in capture._packets]
capture.close()
for packet in packets:
   # do something with the packet
```

<i><b>LiveCapture with BPF_Filter</b></i>

<p align="justify">
The PyShark LiveCapture mode has a <i><b>BPF_Filter</b></i> that allows you to prefilter the packets being captured. The example below show how to parse Domain Name System (DNS) packets from a LiveCapture session.
</p>

```python
capture = pyshark.LiveCapture(interface='your capture interface', bpf_filter='udp port 53')
for packet in capture:
   # do something with the raw_packet
```

<p align="justify">
The FileCapture mode of PyShark also has prefilter capabilities via the <i><b>Display_Filter</b></i>. The example below show how to parse Domain Name System (DNS) packets from a FileCapture session.
</p>

<i><b>FileCapture Display_Filter</b></i>
 
 ```python
 capture = pyshark.FileCapture(pcap_file, display_filter='dns')
 for raw_packet in capture:
    # do something with the raw_packet
```

<i><b>Function Level Filtering</b></i>

This type of packet filtering does not use the built-in PyShark's functions BPF_Filter or Display_Filter.<br>

```python
if hasattr(packet, 'udp') and packet[packet.transport_layer].dstport == '53':
```
or

```python
if hasattr(packet, 'tcp'):
  if packet[packet.transport_layer].dstport == '80' or packet[packet.transport_layer].dstport == '443':
```
</p>

### Accessing packet data elements:
<p align="justify">
All packets have layers, but these layers vary based on the packet type. These layers can be queried and the data elements within these layers can be extracted. Layer types can be accessed using the following parameter:
<br>
  
```
packet.layers
```

<b>Common Layers:</b>
<br>
* ETH Layer - Ethernet
* IP Layer - Internet Protocol
* TCP Layer - Transmission Control Protocol
* UDP Layer - User Datagram Protocol
* ARP Layer - Address Resolution Protocol

<b>Other Layers:</b>
<br>
* BROWSER Layer - Web browser
* DATA Layer - Normal data payload of a protocol
* DB-LSP-DISC Layer - Dropbox LAN Sync Discovery
* DHCP Layer - Dynamic Host Configuration Protocol
* HTTP Layer - Hypertext Transfer Protocol
* LLMNR Layer - Link-Local Multicast Name Resolution
* MAILSLOT Layer - Mailslot protocol is part of the SMB protocol family
* MSNMS Layer - Microsoft Network Messenger Service
* NAT-PMP Layer - NAT Port Mapping Protocol
* NBDGM Layer - NetBIOS Datagram Service
* NBNS Layer - NetBIOS Name Service
* SMB Layer - Server Message Block
* SNMP Layer - Simple Network Management Protocol 
* SSDP Layer - Simple Service Discovery Protocol 
* TLS Layer - Transport Layer Security,
* XML Layer - Extensible Markup Language
</p>

### Parsing examples:
<p align="justify">
PyShark has a lot of flexibility to parse various types of information from an individual network packet. Below are some of the items that can be parsed using the transport_layer and IP layer.


<b>Example One:</b>
```
protocol = packet.transport_layer
source_address = packet.ip.src
source_port = packet[packet.transport_layer].srcport
destination_address = packet.ip.dst
destination_port = packet[packet.transport_layer].dstport 
packet_time = packet.sniff_time
packet_timestamp = packet.sniff_timestamp
```

<b>Output Example One:</b>

```python
Protocol type: UDP
Source address: 192.168.3.1
Source port: 53
Destination address: 192.168.3.131
Destination port: 58673
Date and Time: 2011-01-25 13:57:18.356677
Timestamp: 1295981838.356677000
```
</p>

<b>Example Two:</b>

<p align="justify">
This example shows how to access the field elements within the <i>HTTP layer</i>. The code below queries a Packet Capture (PCAP) file for all the URLs within the <i>HTTP layer</i> with the field name <i>request.full_uri</i>.
</p>

```python
cap_file = 'traffic_flows_small.pcap'
capture = pyshark.FileCapture(pcap_file)
for packet in capture:
   if hasattr(packet, 'http'):
     field_names = packet.http._all_fields
     field_values = packet.http._all_fields.values()
     for field_name in field_names:
        for field_value in field_values:
           if field_name == 'http.request.full_uri' and field_value.startswith('http'):
             print(f'{field_value}')
```

<b>Output Example Two:</b>
<br>
```
https://stackoverflow.com/questions/tagged/python
https://stackoverflow.com/questions/tagged/python-3.x
https://stackoverflow.com/search?q=pyshark
```
</p>

### Additional parsing examples:

<p align="justify"> Here are some additional parsing examples that I posted to <b>GitHub Gist</b>.
  
</p>

* <a href="https://gist.github.com/johnbumgarner/b758aa24c768655940cd3352ce2a0921">Extract the conversation header information from a PCAP packet</a>

* <a href="https://gist.github.com/johnbumgarner/166b6371f975c8e0a0aeae2516771039">Extract DNS elements from a PCAP packet</a>

* <a href="https://gist.github.com/johnbumgarner/ff8c463dc668648dd9ffb0a9a9d939bc">Extract the HTTP information from IPv4 and ICMPv6 packets</a>

* <a href="https://gist.github.com/johnbumgarner/9594e36a31bf1e220838160c37bfc7d4">Extract specific IPv6 elements from a PCAP packet</a>


## Prerequisites
<p align="justify">
TShark has to be installed and accessible via your $PATH, which Python queries for PyShark. For this experiment TShark was installed using <b>Homebrew</b>.<br>

The package Wireshark installs the command line utility TShark. The command used to install Wireshark was:<br>

```
brew install wireshark
```   
</p>

## References:

* [PyShark:](https://kiminewt.github.io/pyshark) Is the Python wrapper for TShark, that allows Python packet parsing using wireshark dissectors.

* [TShark:](https://www.wireshark.org/docs/man-pages/tshark.html) TShark is a terminal oriented version of Wireshark designed for capturing and displaying packets when an interactive user interface isn't necessary or available.

* [Wireshark:](https://www.wireshark.org) Wireshark is a network packet analysis tool that captures packets in real time and displays them in a graphic interface.

* [Homebrew:](https://brew.sh) Package Manager for macOS and Linux.

* [Berkeley Packet Filter (BPF) syntax](https://biot.com/capstats/bpf.html)

* [Display Filter syntax](https://wiki.wireshark.org/DisplayFilters)

## Notes:
<p align="justify">
<b>PyShark</b> has limited documentation, so I would highly recommend reviewing the source code in the PyShark GitHub repository. Several of the parameters listed in this README were pulled directly from the source code.
</p>

_The code within this repository is **not** production ready. It was **strictly** designed for experimental testing purposes only._
