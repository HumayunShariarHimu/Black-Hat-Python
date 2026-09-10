# **Python Subnet Scanner with ICMP Packet Analysis**  

This script **scans a subnet** by sending UDP packets to all hosts and then listens for **ICMP responses**, which can indicate whether a host is active but has no open UDP ports.

```python
import socket
import os
import struct
import threading
from ipaddress import ip_address, ip_network
import ctypes

# the book is performing this section using netaddr library
# but this library is no more maintened 
# and is being overlapped in Python 3 from stlib ipaddress lib

# host to listen on
host = "192.168.0.187"

# subnet to target
tgt_subnet = "192.168.0.0/24"

# magic we'll check ICMP responses for
tgt_message = "PYTHONRULES!"

def udp_sender(sub_net, magic_message):
    sender = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    for ip in ip_network(sub_net).hosts():
        sender.sendto(magic_message.encode("utf-8"), (str(ip), 65212))

# the structure has been taken from canonical IP Header definition
# i.e. sites.uclouvain.be/SystInfo/usr/include/netinet/ip.h.html
class IP(ctypes.Structure):
    _fields_= [
        ("ihl",          ctypes.c_ubyte, 4),
        ("version",      ctypes.c_ubyte, 4),
        ("tos",          ctypes.c_ubyte),
        ("len",          ctypes.c_ushort),
        ("id",           ctypes.c_ushort),
        ("offset",       ctypes.c_ushort),
        ("ttl",          ctypes.c_ubyte),
        ("protocol_num", ctypes.c_ubyte),
        ("sum",          ctypes.c_ushort),
        ("src",          ctypes.c_uint32),
        ("dst",          ctypes.c_uint32)
    ]

    def __new__(cls, socket_buffer=None):
        return cls.from_buffer_copy(socket_buffer)
    
    def __init__(self, socket_buffer=None):
        self.socket_buffer = socket_buffer

        # map protocol constats to their names
        self.protocol_map = {1: "ICMP", 6: "TCP", 17:"UDP"}

        # human readable IP addresses
        self.src_address = socket.inet_ntoa(struct.pack("@I", self.src))
        self.dst_address = socket.inet_ntoa(struct.pack("@I", self.dst))

        # human readable protocol
        try:
            self.protocol = self.protocol_map[self.protocol_num]
        except IndexError:
            self.protocol = str(self.protocol_num)

class ICMP(ctypes.Structure):
    _fields_= [
        ("type",         ctypes.c_ubyte),
        ("code",         ctypes.c_ubyte),
        ("checksum",     ctypes.c_ushort),
        ("unused",       ctypes.c_ushort),
        ("next_hop_mtu", ctypes.c_ushort)
    ]

    def __new__(cls, socket_buffer):
        return cls.from_buffer_copy(socket_buffer)

    def __init__(self, socket_buffer):
        self.socket_buffer = socket_buffer

# create a raw socket and bind it to the public interface
if os.name == "nt":
    socket_protocol = socket.IPPROTO_IP # windows OS
else:
    socket_protocol = socket.IPPROTO_ICMP # unix OS

sniffer = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket_protocol)

# i had trouble with my VM here following along the lesson of the book
# sniffer.bind((host, 0))
# so I was able to made it working forwarding to this port
sniffer.bind(("0.0.0.0", 6677))

# keep IP headers in the capture
sniffer.setsockopt(socket.IPPROTO_IP, socket.IP_HDRINCL, 1)

# if we're on Windows we need to send IOCTL to send promiscuos mode
if os.name == "nt":
    sniffer.ioctl(socket.SIO_RCVALL, socket.RCVALL_ON)

# start sending packets
t = threading.Thread(target=udp_sender, args=(tgt_subnet, tgt_message))
t.start()

try:
    while True:
        # read in a single packet
        raw_buffer = sniffer.recvfrom(65535)[0]

        # create IP header from the first 20 bytes of the buffer
        ip_header = IP(raw_buffer[:20])

        print(f"Protocol: {ip_header.protocol}, {ip_header.src_address} -> {ip_header.dst_address}")

        # if it's ICMP we want the details:
        if ip_header.protocol == "ICMP":
            # find where our ICMP packet starts
            offset = ip_header.ihl * 4 
            buf = raw_buffer[offset:offset + ctypes.sizeof(ICMP)]

            # create our ICMP structure
            icmp_header = ICMP(buf)

            print(f"ICMP -> Type: {icmp_header.type}, {icmp_header.code}")

            # now check for the TYPE 3 and CODE 3
            # that indicates a host is up but has no port available to talk to
            if icmp_header.code == 3 and icmp_header.type == 3:

                # check to make sure we are receiving the response that lands on our subnet
                if ip_address(ip_header.src_address) in ip_network(tgt_subnet):
                    # test our magic message
                    if raw_buffer[len(raw_buffer)- len(tgt_message):] == tgt_message:
                        print(f"Host Up: {ip_header.src_address}")

# handle CTRL-C
except KeyboardInterrupt:
    # turn off promiscuos mode if it has been activated
    if os.name == "nt":
        sniffer.ioctl(socket.SIO_RCVALL, socket.RCVALL_OFF)
```
---

## **Code Breakdown**  

### **1. Importing Required Modules**  
```python
import socket
import os
import struct
import threading
from ipaddress import ip_address, ip_network
import ctypes
```
- `socket` - Used for creating raw sockets to send and receive packets.  
- `os` - Determines the operating system (Windows or Unix).  
- `struct` - Handles binary data manipulation.  
- `threading` - Allows sending UDP packets in a separate thread.  
- `ipaddress` - Used to work with IP addresses and subnets.  
- `ctypes` - Defines C-style structures for parsing **IP** and **ICMP** headers.  

---

### **2. Setting Target Variables**  
```python
host = "192.168.0.187"  # Host to listen on
tgt_subnet = "192.168.0.0/24"  # Subnet to scan
tgt_message = "PYTHONRULES!"  # Message embedded in packets for verification
```
- The script will scan the **192.168.0.0/24** subnet.  
- It sends UDP packets containing the **magic message** `PYTHONRULES!` to detect active hosts.  

---

### **3. UDP Sender Function**  
```python
def udp_sender(sub_net, magic_message):
    sender = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    for ip in ip_network(sub_net).hosts():
        sender.sendto(magic_message.encode("utf-8"), (str(ip), 65212))
```
- This function sends **UDP packets** to every host in the target subnet.  
- UDP packets are sent to port **65212**.  

---

### **4. Defining the IP Header Structure**  
```python
class IP(ctypes.Structure):
    _fields_= [
        ("ihl",          ctypes.c_ubyte, 4),
        ("version",      ctypes.c_ubyte, 4),
        ("tos",          ctypes.c_ubyte),
        ("len",          ctypes.c_ushort),
        ("id",           ctypes.c_ushort),
        ("offset",       ctypes.c_ushort),
        ("ttl",          ctypes.c_ubyte),
        ("protocol_num", ctypes.c_ubyte),
        ("sum",          ctypes.c_ushort),
        ("src",          ctypes.c_uint32),
        ("dst",          ctypes.c_uint32)
    ]
```
- This structure defines an **IP header**, including fields such as **TTL, protocol, source, and destination IPs**.  

```python
def __init__(self, socket_buffer=None):
    self.protocol_map = {1: "ICMP", 6: "TCP", 17:"UDP"}
    self.src_address = socket.inet_ntoa(struct.pack("@I", self.src))
    self.dst_address = socket.inet_ntoa(struct.pack("@I", self.dst))
    
    try:
        self.protocol = self.protocol_map[self.protocol_num]
    except IndexError:
        self.protocol = str(self.protocol_num)
```
- Converts **binary IP addresses** to human-readable format.  
- Maps protocol numbers to names (ICMP, TCP, UDP).  

---

### **5. Defining the ICMP Header Structure**  
```python
class ICMP(ctypes.Structure):
    _fields_= [
        ("type",         ctypes.c_ubyte),
        ("code",         ctypes.c_ubyte),
        ("checksum",     ctypes.c_ushort),
        ("unused",       ctypes.c_ushort),
        ("next_hop_mtu", ctypes.c_ushort)
    ]
```
- Defines the **ICMP packet structure**, which includes:  
  - `type` (ICMP message type, e.g., 3 for Destination Unreachable).  
  - `code` (ICMP error code).  
  - `checksum` (error detection).  

---

### **6. Creating and Binding the Raw Socket**  
```python
if os.name == "nt":
    socket_protocol = socket.IPPROTO_IP  # Windows OS
else:
    socket_protocol = socket.IPPROTO_ICMP  # Unix OS

sniffer = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket_protocol)
sniffer.bind(("0.0.0.0", 6677))  # Listen on all interfaces, port 6677
sniffer.setsockopt(socket.IPPROTO_IP, socket.IP_HDRINCL, 1)  # Capture IP headers
```
- A **raw socket** is created to listen for incoming ICMP packets.  
- Binds to **all network interfaces** on port **6677**.  

```python
if os.name == "nt":
    sniffer.ioctl(socket.SIO_RCVALL, socket.RCVALL_ON)  # Enable promiscuous mode on Windows
```
- Enables **promiscuous mode** to capture all network traffic on Windows.  

---

### **7. Sending UDP Packets in a Separate Thread**  
```python
t = threading.Thread(target=udp_sender, args=(tgt_subnet, tgt_message))
t.start()
```
- Starts **sending UDP packets** in a background thread while the main thread listens for ICMP responses.  

---

### **8. Listening for ICMP Responses**  
```python
try:
    while True:
        raw_buffer = sniffer.recvfrom(65535)[0]
        ip_header = IP(raw_buffer[:20])

        print(f"Protocol: {ip_header.protocol}, {ip_header.src_address} -> {ip_header.dst_address}")
```
- **Receives packets** and extracts the **IP header**.  

```python
if ip_header.protocol == "ICMP":
    offset = ip_header.ihl * 4
    buf = raw_buffer[offset:offset + ctypes.sizeof(ICMP)]
    icmp_header = ICMP(buf)
    
    print(f"ICMP -> Type: {icmp_header.type}, {icmp_header.code}")
```
- If the packet is **ICMP**, extract and display **Type and Code**.  

---

### **9. Checking for Host Discovery**  
```python
if icmp_header.code == 3 and icmp_header.type == 3:
```
- **ICMP Type 3, Code 3** means the host is **reachable** but **no port is open**.  
- This means the **host is up** but does not accept the UDP packet.  

```python
if ip_address(ip_header.src_address) in ip_network(tgt_subnet):
    if raw_buffer[len(raw_buffer)-len(tgt_message):] == tgt_message:
        print(f"Host Up: {ip_header.src_address}")
```
- Checks if the response comes from the **target subnet**.  
- Verifies if the **magic message** is present in the response.  

---

### **10. Handling CTRL+C and Disabling Promiscuous Mode**  
```python
except KeyboardInterrupt:
    if os.name == "nt":
        sniffer.ioctl(socket.SIO_RCVALL, socket.RCVALL_OFF)  # Disable promiscuous mode on Windows
```
- Allows **graceful termination** when pressing `CTRL+C`.  
- Turns off **promiscuous mode** on Windows.  

---

## **How to Run the Code**  

### **Prerequisites**  
- **Python 3.x** installed.  
- **Administrator/root privileges** (Raw sockets require elevated permissions).  
- Works on **Windows** and **Unix-based systems (Linux, macOS, etc.)**.  

### **Steps to Run**  

#### **On Windows**  
1. Open **Command Prompt** as Administrator.  
2. Run the script:  
   ```sh
   python scanner.py
   ```

#### **On Linux/macOS**  
1. Open **Terminal**.  
2. Run the script with root privileges:  
   ```sh
   sudo python3 scanner.py
   ```

---

## **Conclusion**  
- This script **scans a subnet**, sends UDP packets, and listens for **ICMP responses**.  
- It **detects active hosts** even if they do not have open ports.  
