# Test a Socket Connection to a Remote Host

This script is designed to test the ability to create a socket and connect to a specified remote host on a specific port (default is port 80).

```python
#!/usr/bin/env python

import socket,sys

host = "domain/ip_address"
port = 80


try:
    mysocket = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    print(mysocket)
    mysocket.settimeout(5)
except socket.error as e:
	print("socket create error: %s" %e)
	sys.exit(1)
	

try:
    mysocket.connect((host,port))
    print(mysocket)

except socket.timeout as e :
	print("Timeout %s" %e)
	sys.exit(1)
except socket.gaierror as e:
	print("connection error to the server:%s" %e)
	sys.exit(1)
except socket.error as e:
	print("Connection error: %s" %e)
	sys.exit(1)
```

#### Code Explanation:

1. **Imports**:
   - `socket`: The module used for creating sockets and connecting to remote hosts.
   - `sys`: Used for handling system-level operations, especially exiting the script if an error occurs.

2. **Host and Port Setup**:
   - `host = "domain/ip_address"`: This is the domain or IP address of the remote host you want to connect to. You need to replace this with the actual domain name or IP address.
   - `port = 80`: This is the port number you want to connect to. In this example, the default port is 80 (typically used for HTTP services).

3. **Creating the Socket**:
   - `mysocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)`: Creates a socket object that uses the Internet protocol (`AF_INET`) and TCP (`SOCK_STREAM`).
   - `mysocket.settimeout(5)`: Sets a timeout of 5 seconds for the socket connection attempt. This means if the connection doesn't succeed within 5 seconds, a timeout exception will be raised.

4. **Error Handling during Socket Creation**:
   - If the socket creation fails for any reason, an exception is caught (`socket.error`), and the error message is printed. The script then exits with `sys.exit(1)` to indicate an error.

5. **Connecting to the Host**:
   - `mysocket.connect((host, port))`: This attempts to connect to the specified host and port.
   - **Exceptions**:
     - **`socket.timeout`**: If the connection attempt times out (i.e., the server doesn't respond in the specified timeout period), a timeout exception is raised.
     - **`socket.gaierror`**: If there is a DNS resolution error (e.g., the domain name cannot be resolved to an IP address), a connection error is raised.
     - **`socket.error`**: If any other connection error occurs (e.g., network issues, server down, etc.), it is caught by this exception.

6. **Output**:
   - If the connection is successful, the socket object (`mysocket`) is printed, showing details about the socket.
   - If any error occurs during either socket creation or connection, an error message is displayed, and the script exits.

#### How to Run the Script:

1. Save the script to a file, e.g., `socket_connection_test.py`.
2. Open a terminal or command prompt.
3. Navigate to the directory where `socket_connection_test.py` is saved.
4. Run the script using Python:
   ```bash
   python socket_connection_test.py
   ```

### Example Output:

If the connection is successful:

```bash
<socket.socket fd=3, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('0.0.0.0', 0), raddr=('93.184.216.34', 80)>
```

If there is a timeout error:

```bash
Timeout timed out
```

If the host name cannot be resolved:

```bash
connection error to the server: [Errno -2] Name or service not known
```

If there is a general connection error:

```bash
Connection error: [Errno 111] Connection refused
```

#### Notes:
- The script is simple and checks basic connectivity to a remote host. It's a useful tool for diagnosing connectivity issues to specific ports.
- You can modify the `host` and `port` variables to test other hosts and ports.
