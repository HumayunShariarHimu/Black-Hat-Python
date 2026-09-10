### **Code:**
```python
import socket
import threading

# Bind IP and port
bind_ip = "0.0.0.0"  # Listen on all available network interfaces
bind_port = 9999  # Port to listen on

# Create a TCP server socket
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Bind the server to the IP and port
server.bind((bind_ip, bind_port))

# Start listening with a backlog of 5 connections
server.listen(5)

print(f"[*] Listening on {bind_ip}:{bind_port}")

# Client handling thread
def handle_client(client_socket):
    """Handles communication with the connected client."""
    request = client_socket.recv(1024)  # Receive data from the client

    print(f"[*] Received: {request.decode('utf-8')}")  # Print the received data

    # Send back an acknowledgment packet
    client_socket.send(b"ACK!")

    print(f"[*] Connection closed with {client_socket.getpeername()}")
    
    client_socket.close()  # Close the client socket

# Main loop to accept incoming connections
while True:
    client, addr = server.accept()  # Accept a new client connection

    print(f"[*] Accepted connection from {addr[0]}:{addr[1]}")

    # Spin up a new thread to handle the client
    client_handler = threading.Thread(target=handle_client, args=(client,))
    client_handler.start()
```

---

### **How the Server Works:**
1. **Creates a TCP server socket** using IPv4 (`AF_INET`) and TCP (`SOCK_STREAM`).
2. **Binds** to `0.0.0.0` (listens on all available interfaces) on port `9999`.
3. **Listens** for incoming connections (`server.listen(5)`, allowing up to 5 clients in the queue).
4. **Accepts new connections** in an infinite loop.
5. **Spawns a new thread** for each connected client, handling communication without blocking new connections.

---

### **How to Run the Server:**
1. Save this script as `tcp_server.py`.
2. Run it using:
   ```bash
   python tcp_server.py
   ```
3. The server will start listening for connections on port `9999`.

### **Testing the Server:**
- Open another terminal and use `telnet`:
  ```bash
  telnet 127.0.0.1 9999
  ```
- Or use a Python client:
  ```python
  import socket

  client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
  client.connect(("127.0.0.1", 9999))
  client.send(b"Hello, Server!")
  response = client.recv(4096)
  print(response.decode())
  ```
