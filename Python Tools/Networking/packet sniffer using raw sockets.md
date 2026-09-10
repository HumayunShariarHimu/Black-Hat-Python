# A simple packet sniffer using raw sockets

This Python script demonstrates how to create a simple **packet sniffer** using raw sockets. The script captures a single packet from the network interface and prints its contents. Below is a detailed explanation of the code, along with comments and instructions on how to run it.



```python

import socket
import os

# host to listen on 
host = "192.168.0.196"

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

# read in a single packet
print(sniffer.recvfrom(65535)) #the book has a typo here, stating 65565

# turn off promiscuos mode if it has been activated
if os.name == "nt":
    sniffer.ioctl(socket.SIO_RCVALL, socket.RCVALL_OFF)
```


---

### Code Explanation

#### Imports and Configuration
```python
import socket  # For creating and managing network sockets
import os  # For OS-related functions

# Host to listen on (change this to your machine's IP address)
host = "192.168.0.196"
```

#### Create a Raw Socket
The script creates a raw socket, which allows capturing packets at a low level (including headers).

```python
# Determine the protocol based on the OS
if os.name == "nt":  # Windows
    socket_protocol = socket.IPPROTO_IP  # Capture all IP packets
else:  # Unix-like systems (Linux, macOS, etc.)
    socket_protocol = socket.IPPROTO_ICMP  # Capture ICMP packets only

# Create a raw socket
sniffer = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket_protocol)
```

#### Bind the Socket
The socket is bound to a specific IP address and port. If binding to the host IP fails, the script binds to all interfaces (`0.0.0.0`) on port `6677`.

---

### How the Code Works

1. **Create a Raw Socket**:
   - A raw socket is created to capture packets at a low level.
   - The protocol is set to `IPPROTO_IP` on Windows (to capture all IP packets) or `IPPROTO_ICMP` on Unix-like systems (to capture ICMP packets only).

2. **Bind the Socket**:
   - The socket is bound to a specific IP address and port. If binding fails, it defaults to all interfaces (`0.0.0.0`) on port `6677`.

3. **Include IP Headers**:
   - The `IP_HDRINCL` option ensures that IP headers are included in the captured packets.

4. **Enable Promiscuous Mode (Windows)**:
   - On Windows, promiscuous mode is enabled to capture all packets on the network.

5. **Capture a Packet**:
   - The script captures a single packet and prints its contents.

6. **Disable Promiscuous Mode (Windows)**:
   - After capturing the packet, promiscuous mode is disabled on Windows.

---

### How to Run the Code

1. **Install Python**:
   - Ensure Python is installed on your system. You can download it from [python.org](https://www.python.org/).

2. **Run the Script**:
   - Save the script to a file, e.g., `packet_sniffer.py`.
   - Run the script with administrative privileges (required for raw socket access):
     ```bash
     sudo python packet_sniffer.py  # On Unix-like systems
     python packet_sniffer.py       # On Windows (run as Administrator)
     ```

3. **Observe the Output**:
   - The script will capture a single packet and print its raw contents, including the IP headers.

---

### Notes

- **Permissions**: Raw socket operations require administrative privileges. On Unix-like systems, use `sudo`. On Windows, run the script as an Administrator.
- **OS Differences**:
  - On Windows, the script captures all IP packets.
  - On Unix-like systems, the script captures only ICMP packets by default.
- **Promiscuous Mode**: Promiscuous mode is only enabled on Windows. On Unix-like systems, you may need additional tools (e.g., `tcpdump`) to capture all packets.
- **Port Binding**: The script binds to port `6677` as a workaround. You can modify this to bind to a different port or interface.

---
