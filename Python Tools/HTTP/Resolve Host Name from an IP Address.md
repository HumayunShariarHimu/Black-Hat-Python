# Resolve Host Name from an IP Address

This script resolves the host name for a given IP address and prints the corresponding IP addresses associated with the host.

```python
#!/usr/bin/env python

import socket

try :
	result = socket.gethostbyaddr("8.8.8.8")
	print("The host name is:",result[0])
	print("Ip addresses:")
	for item in result[2]:
		print(" "+item)
except socket.error as e:
	print("Error for resolving ip address:",e)
```

#### Code Explanation:

1. **Imports**:
   - `socket`: This module provides low-level networking interfaces, such as resolving hostnames and working with IP addresses.

2. **Resolving the Host Name**:
   - `socket.gethostbyaddr("8.8.8.8")`: This function is used to resolve the host name for the given IP address (`8.8.8.8` in this case). It returns a tuple containing:
     - The primary host name.
     - An alias list (which can be empty or contain alternative names for the host).
     - A list of IP addresses associated with the host.
   
   - **Result**: 
     - `result[0]`: The primary host name (e.g., "dns.google").
     - `result[2]`: The list of IP addresses associated with the host (e.g., `['8.8.8.8', '8.8.4.4']`).

3. **Printing the Results**:
   - The primary host name (`result[0]`) is printed.
   - A loop iterates through `result[2]`, which contains the IP addresses associated with the host, and each IP address is printed.

4. **Error Handling**:
   - If there is any error (e.g., the IP address cannot be resolved or is invalid), a `socket.error` exception is caught, and an error message is printed.

#### How to Run the Script:

1. Save the script to a file, e.g., `resolve_host.py`.
2. Open a terminal or command prompt.
3. Navigate to the directory where `resolve_host.py` is saved.
4. Run the script using Python:
   ```bash
   python resolve_host.py
   ```

### Example Output:

For the IP address `8.8.8.8` (Google DNS):

```bash
The host name is: dns.google
Ip addresses:
 8.8.8.8
 8.8.4.4
```

If there is an error (e.g., invalid IP):

```bash
Error for resolving ip address: [Errno 11004] getaddrinfo failed
```

#### Notes:
- You can replace the IP address `8.8.8.8` with any valid IP address to resolve its host name and associated IP addresses.
- This script is useful for DNS reverse lookups, helping to find the host name corresponding to an IP address.
