# TCP proxy 

### **TCP Proxy:**
```python
import sys
import socket
import threading

def hexdump(src, length=16):
    """Hex Dump
    This function produces a classic 3-column hex dump of a string.
    The first column prints the offset in hexadecimal.
    The second column prints the hexadecimal byte values.
    The third column prints ASCII values or a dot (.) for non-printable characters.
    """
    result = []
    digits = 4  # Offset width

    if isinstance(src, bytes):  # Ensure we handle bytes properly
        for i in range(0, len(src), length):
            s = src[i:i+length]
            hexa = " ".join(f"{x:02X}" for x in s)  # Hex representation
            text = "".join(chr(x) if 0x20 <= x < 0x7F else "." for x in s)  # ASCII values
            result.append(f"{i:04X}  {hexa:<{length*3}}  {text}")

    print("\n".join(result))


def receive_from(connection):
    """Receive data from the socket.
    Reads data until no more is available or a timeout occurs.
    """
    buffer = b""
    connection.settimeout(2)

    try:
        while True:
            data = connection.recv(4096)
            if not data:
                break
            buffer += data
    except socket.timeout:
        pass

    return buffer


def request_handler(buffer):
    """Modify requests before sending them to the remote host (if needed)."""
    return buffer


def response_handler(buffer):
    """Modify responses before sending them back to the local client (if needed)."""
    return buffer


def proxy_handler(client_socket, remote_host, remote_port, receive_first):
    """Handles communication between the local client and remote server."""
    
    # Create a connection to the remote host
    remote_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    remote_socket.connect((remote_host, remote_port))

    # If we need to receive data first from the remote server
    if receive_first:
        remote_buffer = receive_from(remote_socket)
        if remote_buffer:
            print("[<==] Received from remote:")
            hexdump(remote_buffer)

            # Send data to response handler
            remote_buffer = response_handler(remote_buffer)

            # Forward modified response to the local client
            client_socket.send(remote_buffer)

    while True:
        # Receive data from local client
        local_buffer = receive_from(client_socket)
        if local_buffer:
            print(f"[==>] Received {len(local_buffer)} bytes from localhost")
            hexdump(local_buffer)

            # Modify the request if needed
            local_buffer = request_handler(local_buffer)

            # Forward the request to the remote server
            remote_socket.send(local_buffer)
            print("[==>] Sent to remote")

        # Receive response from remote server
        remote_buffer = receive_from(remote_socket)
        if remote_buffer:
            print(f"[<==] Received {len(remote_buffer)} bytes from remote")
            hexdump(remote_buffer)

            # Modify the response if needed
            remote_buffer = response_handler(remote_buffer)

            # Forward the response to the local client
            client_socket.send(remote_buffer)
            print("[<==] Sent to localhost")

        # Close connections when no more data
        if not local_buffer or not remote_buffer:
            client_socket.close()
            remote_socket.close()
            print("[*] No more data. Closing connections.")
            break


def server_loop(local_host, local_port, remote_host, remote_port, receive_first):
    """Creates a listening server that forwards traffic to a remote server."""
    
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
    try:
        server.bind((local_host, local_port))
    except socket.error as e:
        print(f"[!!] Failed to bind to {local_host}:{local_port}")
        print(f"[!!] Ensure no other processes are using the port.")
        print(f"Error: {e}")
        sys.exit(1)

    print(f"[*] Listening on {local_host}:{local_port}")

    server.listen(5)

    while True:
        client_socket, addr = server.accept()
        print(f"[==>] Incoming connection from {addr[0]}:{addr[1]}")

        # Create a new thread to handle the connection
        proxy_thread = threading.Thread(target=proxy_handler, args=(client_socket, remote_host, remote_port, receive_first))
        proxy_thread.start()


def main():
    """Main function to parse arguments and start the proxy server."""
    
    if len(sys.argv[1:]) != 5:
        print("Usage: ./tcp_proxy.py [localhost] [localport] [remotehost] [remoteport] [receive_first]")
        print("Example: ./tcp_proxy.py 127.0.0.1 9000 10.12.132.1 9000 True")
        sys.exit(1)

    # Assign command-line arguments
    local_host = sys.argv[1]
    local_port = int(sys.argv[2])
    remote_host = sys.argv[3]
    remote_port = int(sys.argv[4])

    # Convert string "True" or "False" to boolean
    receive_first = sys.argv[5].lower() == "true"

    # Start the server loop
    server_loop(local_host, local_port, remote_host, remote_port, receive_first)


if __name__ == "__main__":
    main()
```

---

### **How This Proxy Works**
1. **Listens on a local port** and waits for connections.
2. **Forwards incoming traffic** to a remote host and port.
3. **Receives responses** from the remote host.
4. **Passes traffic through handlers** (`request_handler` and `response_handler`), allowing modifications.
5. **Sends modified data back** to the local client.

---

### **Fixes & Improvements**
- **Fixed `hexdump` Function**: Now correctly processes byte sequences.
- **Improved Exception Handling**: More specific error messages.
- **Fixed `TimeoutError` Handling**: Used `socket.timeout` instead.
- **Enhanced Readability**: Added comments and clarified logic.

---

### **Running the Script**
```sh
python3 tcp_proxy.py 127.0.0.1 9000 example.com 80 True
```
- This sets up a proxy that listens on `127.0.0.1:9000` and forwards traffic to `example.com:80`.
- The `True` argument means it will **receive** data first from `example.com` before forwarding the client’s request.

---

### **Possible Use Cases**
✔️ **Traffic Inspection**: View raw data between client and server.  
✔️ **Modifying Requests/Responses**: Test security vulnerabilities.  
✔️ **Debugging Network Issues**: Understand network behavior.

