## **Basic TCP Client in Python**  

This script demonstrates how to create a simple TCP client using Python's `socket` module. The client connects to `www.google.com` on port `80` (HTTP) and sends a basic HTTP `GET` request. The server's response is then received and printed.  

### **How to Run the Code:**  
1. Ensure you have Python installed (`Python 3.x` recommended).  
2. Copy and paste the script into a Python file (e.g., `tcp_client.py`).  
3. Run the script using the command:  
   ```bash
   python tcp_client.py
   ```

---

### **Python Code:**
```python
import socket  # Import the socket module to enable network communication

# Define target server and port
target_host = "www.google.com"  # The website to connect to
target_port = 80  # HTTP port (80 for web traffic)

# Create a socket object using:
# AF_INET -> IPv4
# SOCK_STREAM -> TCP (Transmission Control Protocol)
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connect the client to the target server and port
client.connect((target_host, target_port))

# Send an HTTP GET request to fetch the homepage
# The request follows HTTP/1.1 format and includes the Host header
client.send(b"GET / HTTP/1.1\r\nHost: google.com\r\n\r\n")

# Receive the server's response (up to 4096 bytes)
response = client.recv(4096)

# Print the received response from Google
print(response)
```

---

### **Explanation of Key Concepts:**  
1. **Sockets:** Sockets enable network communication between devices.  
2. **TCP Connection:**  
   - `AF_INET` specifies that we're using IPv4.  
   - `SOCK_STREAM` specifies TCP (which ensures reliable communication).  
3. **HTTP Request:** We manually send a simple HTTP `GET` request.  
4. **Receiving Data:** The response is received in chunks (4096 bytes).  

**Note:** Google may block direct connections from such scripts. To test on a local server, change `target_host` to `"localhost"` and `target_port` to `8080` (or any running server port).
