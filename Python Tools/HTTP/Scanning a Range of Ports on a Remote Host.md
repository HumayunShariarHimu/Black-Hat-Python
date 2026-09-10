# Scanning a Range of Ports on a Remote Host

This script allows you to scan a range of ports on a remote host and check if they are open or closed.

```python
#!/usr/bin/env python

import socket
import sys
from datetime import datetime
import errno

remoteServer    = input("Enter a remote host to scan: ")
remoteServerIP  = socket.gethostbyname(remoteServer)

print("Please enter the range of ports you would like to scan on the machine")
startPort    = input("Enter a start port: ")
endPort    = input("Enter a end port: ")

print("Please wait, scanning remote host", remoteServerIP)

time_init = datetime.now()

try:
	for port in range(int(startPort),int(endPort)):
		print ("Checking port {} ...".format(port))
		sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		sock.settimeout(5)
		result = sock.connect_ex((remoteServerIP, port))
		if result == 0:
			print("Port {}: 	 Open".format(port))
		else:
			print("Port {}: 	 Closed".format(port))
			print("Reason:",errno.errorcode[result])
		sock.close()

except KeyboardInterrupt:
	print("You pressed Ctrl+C")
	sys.exit()
except socket.gaierror:
	print('Hostname could not be resolved. Exiting')
	sys.exit()
except socket.error:
	print("Couldn't connect to server")
	sys.exit()

time_finish = datetime.now()
total =  time_finish - time_init
print('Port Scanning Completed in: ', total)
```


#### Code Explanation:

1. **Imports**:
   - `socket`: This module provides low-level networking interfaces to create sockets and connect to remote servers.
   - `sys`: Used for interacting with the system, especially for handling exceptions and exiting the program.
   - `datetime`: This module is used to track the start and end times of the scanning process, so the script can output how long the scan took.
   - `errno`: Provides standard error codes for socket-related issues (although this part is not necessary in this context, it’s intended for better error messages).

2. **User Input**:
   - `remoteServer`: Asks the user to input the host or domain name of the machine to scan.
   - `remoteServerIP`: Resolves the domain name (`remoteServer`) into an IP address using `socket.gethostbyname`.
   - `startPort` and `endPort`: Asks the user for a range of ports to scan (inclusive of both start and end).

3. **Time Tracking**:
   - `time_init` is initialized at the start of the scanning process to calculate how long the scan takes.
   - `time_finish` is recorded at the end, and the difference gives the total time for the scan.

4. **Port Scanning Logic**:
   - The script loops through all ports in the specified range (`startPort` to `endPort`).
   - For each port, a socket object is created (`sock`), and `sock.settimeout(5)` sets a 5-second timeout for the connection attempt.
   - The `sock.connect_ex()` method is used to try to connect to the target IP address and port. If it returns `0`, the port is open. If any other value is returned, the port is closed.
   - If the port is closed, the script prints the error code from `errno.errorcode[result]`, which helps understand the specific connection failure reason.
   - After checking each port, the socket is closed (`sock.close()`).

5. **Exception Handling**:
   - **KeyboardInterrupt**: If the user presses `Ctrl+C`, the program will exit gracefully with a message.
   - **socket.gaierror**: If the hostname can't be resolved (e.g., incorrect or unreachable host), it prints an error and exits.
   - **socket.error**: If there's any error connecting to the server, the program will print an error message and exit.

6. **Final Output**:
   - After scanning, the script calculates and prints how long the scanning process took by subtracting `time_init` from `time_finish`.

#### How to Run the Script:

1. Save the script to a file, e.g., `port_scanner.py`.
2. Open a terminal or command prompt.
3. Navigate to the directory where `port_scanner.py` is saved.
4. Run the script using the Python interpreter:
   ```bash
   python port_scanner.py
   ```

### Example Output:

```bash
Enter a remote host to scan: example.com
Please enter the range of ports you would like to scan on the machine
Enter a start port: 20
Enter a end port: 25
Please wait, scanning remote host 93.184.216.34
Checking port 20 ...
Port 20:      Closed
Reason: [Errno 111] Connection refused
Checking port 21 ...
Port 21:      Open
Checking port 22 ...
Port 22:      Open
Checking port 23 ...
Port 23:      Closed
Reason: [Errno 111] Connection refused
...
Port Scanning Completed in:  0:00:12.345678
```

#### Notes:
- The user is prompted for the host and the port range, making it flexible.
- The script checks multiple ports sequentially and reports the status of each one.
- The timeout is set to 5 seconds to avoid long waits for unresponsive ports. You can adjust this value if needed.
