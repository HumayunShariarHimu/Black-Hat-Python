# **Python Packet Sniffer Using Raw Sockets**  

This Python script is a simple packet sniffer that captures and inspects network packets at a low level using raw sockets. It is designed to work on both Windows and Unix-based systems.  

```python
import socket
import os
import struct
import ctypes

# host to listen on
host = "192.168.0.187"

# the structure has been taken from canonical IP Header definition
# i.e. sites.uclouvain.be/SystInfo/usr/include/netinet/ip.h.html
class IP(ctypes.Structure):
    _fields_= [
        ("ihl", ctypes.c_ubyte, 4),
        ("version", ctypes.c_ubyte, 4),
        ("tos", ctypes.c_ubyte),
        ("len", ctypes.c_ushort),
        ("id", ctypes.c_ushort),
        ("offset", ctypes.c_ushort),
        ("ttl", ctypes.c_ubyte),
        ("protocol_num", ctypes.c_ubyte),
        ("sum", ctypes.c_ushort),
        ("src", ctypes.c_uint32),
        ("dst", ctypes.c_uint32)
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

try:
    while True:
        # read in a single packet
        raw_buffer = sniffer.recvfrom(65535)[0]

        # create IP header from the first 20 bytes of the buffer
        ip_header = IP(raw_buffer[:20])

        print(f"Protocol: {ip_header.protocol}, {ip_header.src_address} -> {ip_header.dst_address}")

# handle CTRL-C
except KeyboardInterrupt:
    # turn off promiscuos mode if it has been activated
    if os.name == "nt":
        sniffer.ioctl(socket.SIO_RCVALL, socket.RCVALL_OFF)
```

## **Code Explanation**  

### **1. Importing Required Modules**  
```python
import socket
import os
import struct
import ctypes
```
- `socket` - Used to create raw sockets for packet sniffing.  
- `os` - Determines the operating system type.  
- `struct` - Handles binary data conversion.  
- `ctypes` - Defines a C-style structure for parsing IP headers.  

---

### **2. Defining the Host to Listen On**  
```python
host = "192.168.0.187"
```
- This is the IP address of the machine running the script, which will be used to listen for incoming packets.  

---

### **3. Defining the IP Header Structure**  
```python
class IP(ctypes.Structure):
    _fields_= [
        ("ihl", ctypes.c_ubyte, 4),
        ("version", ctypes.c_ubyte, 4),
        ("tos", ctypes.c_ubyte),
        ("len", ctypes.c_ushort),
        ("id", ctypes.c_ushort),
        ("offset", ctypes.c_ushort),
        ("ttl", ctypes.c_ubyte),
        ("protocol_num", ctypes.c_ubyte),
        ("sum", ctypes.c_ushort),
        ("src", ctypes.c_uint32),
        ("dst", ctypes.c_uint32)
    ]
```
- This structure represents an IP header, as defined in the TCP/IP protocol stack.  
- It includes fields such as **version, header length, total length, ID, TTL (Time to Live), protocol, source, and destination IP addresses**.  

---

### **4. Converting Binary Data to Readable Format**  
```python
def __new__(cls, socket_buffer=None):
    return cls.from_buffer_copy(socket_buffer)
```
- This allows the class to be instantiated from raw socket data.  

```python
def __init__(self, socket_buffer=None):
    self.socket_buffer = socket_buffer
    self.protocol_map = {1: "ICMP", 6: "TCP", 17:"UDP"}
```
- Maps protocol numbers (1, 6, 17) to their respective names (`ICMP`, `TCP`, `UDP`).  

```python
self.src_address = socket.inet_ntoa(struct.pack("@I", self.src))
self.dst_address = socket.inet_ntoa(struct.pack("@I", self.dst))
```
- Converts the source and destination IP addresses from binary to human-readable format.  

---

### **5. Creating a Raw Socket**  
```python
if os.name == "nt":
    socket_protocol = socket.IPPROTO_IP # Windows
else:
    socket_protocol = socket.IPPROTO_ICMP # Unix-based systems

sniffer = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket_protocol)
```
- Creates a raw socket for capturing packets.  
- Uses `IPPROTO_IP` for Windows and `IPPROTO_ICMP` for Unix-based systems.  

---

### **6. Binding the Sniffer to an Interface**  
```python
sniffer.bind(("0.0.0.0", 6677))
```
- Binds the sniffer to listen on **all network interfaces** (`0.0.0.0`) and port `6677`.  

```python
sniffer.setsockopt(socket.IPPROTO_IP, socket.IP_HDRINCL, 1)
```
- Ensures that captured packets include the IP headers.  

---

### **7. Enabling Promiscuous Mode on Windows**  
```python
if os.name == "nt":
    sniffer.ioctl(socket.SIO_RCVALL, socket.RCVALL_ON)
```
- On Windows, enables **promiscuous mode**, allowing the capture of all network traffic instead of just traffic intended for the host.  

---

### **8. Capturing and Processing Packets**  
```python
try:
    while True:
        raw_buffer = sniffer.recvfrom(65535)[0]  # Read a single packet
        ip_header = IP(raw_buffer[:20])  # Extract IP header (first 20 bytes)

        print(f"Protocol: {ip_header.protocol}, {ip_header.src_address} -> {ip_header.dst_address}")
```
- Listens indefinitely for incoming packets.  
- Extracts and decodes the **IP header**.  
- Displays the **protocol type, source IP, and destination IP**.  

---

### **9. Handling Interrupt (CTRL+C) and Disabling Promiscuous Mode**  
```python
except KeyboardInterrupt:
    if os.name == "nt":
        sniffer.ioctl(socket.SIO_RCVALL, socket.RCVALL_OFF)
```
- Gracefully stops the program when `CTRL+C` is pressed.  
- Turns off **promiscuous mode** on Windows before exiting.  

---

## **Conclusion**  
- This script allows network administrators and security researchers to **analyze incoming packets** at the raw level.  
- It demonstrates **how to construct and parse IP headers** and how to use **raw sockets** for network monitoring.  
- This can be extended to capture **TCP, UDP, and ICMP packets**, making it a useful tool for network security analysis.
