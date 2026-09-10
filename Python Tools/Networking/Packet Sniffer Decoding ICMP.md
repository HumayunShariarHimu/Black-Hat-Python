# **Python Packet Sniffer with ICMP Packet Parsing**  

This script is an advanced **packet sniffer** that captures and analyzes network packets using **raw sockets**. It processes **IP packets** and extracts **ICMP (Internet Control Message Protocol) packets**, displaying relevant information about them.  

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

# handle CTRL-C
except KeyboardInterrupt:
    # turn off promiscuos mode if it has been activated
    if os.name == "nt":
        sniffer.ioctl(socket.SIO_RCVALL, socket.RCVALL_OFF)
```

## **Explanation**  

### **1. Importing Required Modules**  
```python
import socket
import os
import struct
import ctypes
```
- `socket` - Used to create raw sockets for capturing network packets.  
- `os` - Determines the operating system (Windows or Unix).  
- `struct` - Handles binary data conversion.  
- `ctypes` - Used to define C-style structures for parsing IP and ICMP headers.  

---

### **2. Defining the Host to Listen On**  
```python
host = "192.168.0.187"
```
- This is the IP address where the script will listen for incoming packets.  

---

### **3. Defining the IP Header Structure**  
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
- This structure represents an **IP header**, containing fields such as **version, header length, TTL, protocol, source, and destination IP addresses**.  

---

### **4. Converting Binary Data to Readable Format**  
```python
def __new__(cls, socket_buffer=None):
    return cls.from_buffer_copy(socket_buffer)
```
- This allows the class to be instantiated directly from raw socket data.  

```python
def __init__(self, socket_buffer=None):
    self.protocol_map = {1: "ICMP", 6: "TCP", 17: "UDP"}
    self.src_address = socket.inet_ntoa(struct.pack("@I", self.src))
    self.dst_address = socket.inet_ntoa(struct.pack("@I", self.dst))

    try:
        self.protocol = self.protocol_map[self.protocol_num]
    except IndexError:
        self.protocol = str(self.protocol_num)
```
- **Maps protocol numbers** to their respective names (`ICMP`, `TCP`, `UDP`).  
- Converts **binary IP addresses** to human-readable format.  

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
  - `type` (ICMP type, e.g., 8 for Echo Request, 0 for Echo Reply).  
  - `code` (ICMP code, used for error handling).  
  - `checksum` (used for error detection).  
  - `unused` and `next_hop_mtu` (additional fields used in certain ICMP messages).  

---

### **6. Creating and Binding the Raw Socket**  
```python
if os.name == "nt":
    socket_protocol = socket.IPPROTO_IP  # Windows OS
else:
    socket_protocol = socket.IPPROTO_ICMP  # Unix OS

sniffer = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket_protocol)
```
- Creates a **raw socket** to capture network packets.  
- Uses `IPPROTO_IP` for Windows and `IPPROTO_ICMP` for Unix-based systems.  

```python
sniffer.bind(("0.0.0.0", 6677))
```
- Binds the socket to **listen on all interfaces** (`0.0.0.0`) and **port 6677**.  

```python
sniffer.setsockopt(socket.IPPROTO_IP, socket.IP_HDRINCL, 1)
```
- Ensures that the socket captures **IP headers**.  

---

### **7. Enabling Promiscuous Mode on Windows**  
```python
if os.name == "nt":
    sniffer.ioctl(socket.SIO_RCVALL, socket.RCVALL_ON)
```
- Enables **promiscuous mode** on Windows, allowing the capture of all network traffic.  

---

### **8. Capturing and Processing Packets**  
```python
try:
    while True:
        raw_buffer = sniffer.recvfrom(65535)[0]  # Read a single packet
        ip_header = IP(raw_buffer[:20])  # Extract IP header (first 20 bytes)

        print(f"Protocol: {ip_header.protocol}, {ip_header.src_address} -> {ip_header.dst_address}")
```
- **Reads incoming packets** and extracts the **IP header**.  
- Displays the **protocol, source, and destination IP addresses**.  

---

### **9. Extracting ICMP Packet Details**  
```python
if ip_header.protocol == "ICMP":
    offset = ip_header.ihl * 4
    buf = raw_buffer[offset:offset + ctypes.sizeof(ICMP)]

    icmp_header = ICMP(buf)
    print(f"ICMP -> Type: {icmp_header.type}, {icmp_header.code}")
```
- Checks if the **packet is an ICMP packet**.  
- Extracts **ICMP-specific fields** such as **Type and Code**.  

---

### **10. Handling Interrupt (CTRL+C) and Disabling Promiscuous Mode**  
```python
except KeyboardInterrupt:
    if os.name == "nt":
        sniffer.ioctl(socket.SIO_RCVALL, socket.RCVALL_OFF)
```
- Gracefully stops the script when `CTRL+C` is pressed.  
- Turns off **promiscuous mode** on Windows before exiting.  

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
   python sniffer.py
   ```

#### **On Linux/macOS**  
1. Open **Terminal**.  
2. Run the script with root privileges:  
   ```sh
   sudo python3 sniffer.py
   ```

---

## **Expected Output Example**  

```sh
Protocol: ICMP, 192.168.0.1 -> 192.168.0.187
ICMP -> Type: 8, Code: 0
Protocol: TCP, 192.168.0.50 -> 192.168.0.187
Protocol: UDP, 192.168.0.33 -> 192.168.0.187
```
- Captures **ICMP, TCP, and UDP packets**.  
- If it's an **ICMP packet**, it also prints the **ICMP type and code**.  

---

## **Conclusion**  
- This script **sniffs and analyzes network traffic** at a low level.  
- It captures **IP packets**, extracts **ICMP details**, and provides **real-time packet inspection**.  
- This is useful for **network monitoring, security analysis, and debugging**.  
