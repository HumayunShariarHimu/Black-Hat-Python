 # Multithreaded TCP Port Scanning

This script allows you to perform a multi-threaded TCP port scan on a remote host, checking the status of specified ports (open or closed).



```python
#!/usr/bin/python

import optparse
from socket import *
from threading import *

def socketScan(host, port):
	try:
		socket_connect = socket(AF_INET, SOCK_STREAM)
		socket_connect.settimeout(5)
		result = socket_connect.connect((host, port))
		print('[+] %d/tcp open' % port)
	except Exception as exception:
		print('[-] %d/tcp closed' % port)
		print('[-] Reason:%s' % str(exception))
	finally:
		socket_connect.close()	

def portScanning(host, ports):
	try:
		ip = gethostbyname(host)
		print('[+] Scan Results for: ' + ip)
	except:
		print("[-] Cannot resolve '%s': Unknown host" %host)
		return

	for port in ports:
		t = Thread(target=socketScan,args=(ip,int(port)))
		t.start()

def main():
	parser = optparse.OptionParser('socket_portScan '+ '-H <Host> -P <Port>')
	parser.add_option('-H', dest='host', type='string', help='specify host')
	parser.add_option('-P', dest='port', type='string', help='specify port[s] separated by comma')

	(options, args) = parser.parse_args()

	host = options.host
	ports = str(options.port).split(',')

	if (host == None) | (ports[0] == None):
		print(parser.usage)
		exit(0)

	portScanning(host, ports)


if __name__ == '__main__':
	main()
```

#### Code Explanation:

1. **Imports**:
   - `optparse`: This module is used to parse command-line options and arguments.
   - `socket`: Provides low-level networking interfaces to create sockets and connect to remote servers.
   - `threading`: This module is used to create multiple threads for concurrent execution.

2. **`socketScan` Function**:
   - **Parameters**: 
     - `host`: The target host (IP or domain).
     - `port`: The port to be checked.
   - **Process**:
     - This function creates a socket, connects to the specified `host` and `port`, and checks whether the port is open or closed.
     - If the connection succeeds, it prints that the port is open.
     - If the connection fails, it catches the exception and prints that the port is closed, along with the error reason.
   - The socket is closed in the `finally` block to ensure the connection is properly terminated.

3. **`portScanning` Function**:
   - **Parameters**: 
     - `host`: The target host (IP or domain).
     - `ports`: A list of ports to scan.
   - **Process**:
     - Resolves the `host` name to its corresponding IP address using `gethostbyname`.
     - If the host cannot be resolved, it prints an error message and returns.
     - For each port in the list, it creates a new thread (`Thread`) to run the `socketScan` function concurrently, allowing for faster scanning of multiple ports.

4. **`main` Function**:
   - **Command-Line Arguments**:
     - `-H <Host>`: Specifies the target host (IP or domain).
     - `-P <Port(s)>`: Specifies a list of ports (comma-separated) to scan.
   - **Process**:
     - The function uses `optparse` to parse the command-line arguments.
     - It then calls `portScanning` with the host and ports provided by the user.

5. **Main Execution**:
   - If the script is run directly (`if __name__ == '__main__':`), it calls the `main()` function to initiate the port scanning process.

#### How to Run the Script:

1. Save the script to a file, e.g., `port_scanner.py`.
2. Open a terminal or command prompt.
3. Navigate to the directory where `port_scanner.py` is saved.
4. Run the script using the Python interpreter:
   ```bash
   python port_scanner.py -H <target_host> -P <port1,port2,...>
   ```
   Replace `<target_host>` with the IP address or domain name of the host you want to scan, and `<port1,port2,...>` with a comma-separated list of ports to check. For example:
   ```bash
   python port_scanner.py -H 192.168.1.1 -P 22,80,443
   ```

### Example Command:
```bash
python port_scanner.py -H example.com -P 21,22,80,443,8080
```

### Example Output:
```bash
[+] Scan Results for: 93.184.216.34
[+] 21/tcp open
[+] 22/tcp open
[-] 80/tcp closed
[-] Reason:[Errno 111] Connection refused
[+] 443/tcp open
[+] 8080/tcp open
```

#### Notes:
- The script uses threading to check multiple ports concurrently, improving the scan speed.
- The script can be further optimized or extended by adding features such as scanning a range of ports or saving the scan results to a file.
