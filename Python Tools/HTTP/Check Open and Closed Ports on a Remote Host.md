## Check Open and Closed Ports on a Remote Host

### Code Explanation:

This Python script uses the `socket` library to create a socket connection and checks if specified ports are open on the given IP address (in this case, `localhost`). The script will display whether each port is open or closed.

```python
#!/usr/bin/python

import socket
import sys

def checkPortsSocket(ip, portlist):
    try:
        # Loop through each port in the portlist
        for port in portlist:
            # Create a socket object
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(5)  # Set a timeout of 5 seconds for each connection attempt
            
            # Try to connect to the IP address and port
            result = sock.connect_ex((ip, port))
            
            # If result == 0, the port is open
            if result == 0:
                print("Port {}: \t Open".format(port))
            else:
                print("Port {}: \t Closed".format(port))
            
            sock.close()  # Close the socket after checking each port

    except socket.error as error:
        # Handle any connection errors
        print(str(error))
        print("Connection error")
        sys.exit()

# Call the function with localhost and a list of ports to check
checkPortsSocket('localhost', [21, 22, 80, 8080, 443])
```

#### Code Explanation:
1. **Imports**:
   - `socket`: This module provides access to the BSD socket interface, which includes creating sockets and connecting to remote servers.
   - `sys`: This module is used to interact with the system, especially for exiting the program in case of an error.

2. **`checkPortsSocket` Function**:
   - **Parameters**:
     - `ip`: The IP address (or hostname) of the machine you want to check.
     - `portlist`: A list of port numbers to check.
   
   - **Process**:
     - The function loops through the provided list of ports and attempts to establish a TCP connection to each port using the `socket.socket` method.
     - `sock.settimeout(5)` sets a timeout of 5 seconds for each connection attempt.
     - `sock.connect_ex((ip, port))` tries to connect to the specified IP and port. If the connection is successful, `connect_ex` will return 0, indicating that the port is open.
     - Based on the result, the script will print whether each port is open or closed.
     - After checking each port, the socket is closed using `sock.close()`.

3. **Error Handling**:
   - If any socket error occurs, it prints the error message and exits the program using `sys.exit()`.

4. **Main Execution**:
   - The function `checkPortsSocket('localhost', [21, 22, 80, 8080, 443])` is called, which checks the status of ports `21`, `22`, `80`, `8080`, and `443` on the localhost (`127.0.0.1`).

#### How to Run the Script:
1. Save the script to a file, e.g., `check_ports.py`.
2. Open a terminal or command prompt.
3. Navigate to the directory where `check_ports.py` is saved.
4. Run the script using the Python interpreter:
   ```bash
   python check_ports.py
   ```
   This will check the specified ports on `localhost` and print whether each port is open or closed.

### Example Output:
```bash
Port 21:      Closed
Port 22:      Open
Port 80:      Open
Port 8080:    Closed
Port 443:     Open
```


This simple Python script can be used to check the status of various ports on a machine. It helps in identifying whether critical services like SSH (port 22), HTTP (port 80), HTTPS (port 443), or FTP (port 21) are accessible. You can modify this script to fit your specific needs, such as checking different ports or working with remote IP addresses instead of `localhost`.
