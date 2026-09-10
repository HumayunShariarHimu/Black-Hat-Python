This Python script demonstrates how to create a basic SSH server using the `paramiko` library. The server listens for incoming SSH connections, authenticates users, and allows them to execute commands remotely.

---

### Code

```python
import socket  # For creating and managing network sockets
import paramiko  # For implementing SSH functionality
import threading  # For handling threading (used in the Server class)
import sys  # For system-specific parameters and functions

# Load the server's host key from a file (used for SSH authentication)
host_key = paramiko.RSAKey(filename="test_rsa.key")

# Define a custom SSH server class by inheriting from paramiko.ServerInterface
class Server(paramiko.ServerInterface):
    def __init__(self):
        self.event = threading.Event()  # Used for thread synchronization

    # Check if the requested channel type is allowed
    def check_channel_request(self, kind, chanid):
        if kind == "session":  # Allow only session channels
            return paramiko.OPEN_SUCCEEDED
        return paramiko.OPEN_FAILED_ADMINISTRATIVELY_PROHIBITED

    # Authenticate users based on username and password
    def check_auth_password(self, username, password):
        if username == "root" and password == "toor":  # Hardcoded credentials
            return paramiko.AUTH_SUCCESSFUL
        return paramiko.AUTH_FAILED

# Get server IP and port from command-line arguments
server = sys.argv[1]  # Server IP address
ssh_port = int(sys.argv[2])  # SSH port number

try:
    # Create a TCP socket
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  # Allow reuse of the address
    sock.bind((server, ssh_port))  # Bind the socket to the server IP and port
    sock.listen(100)  # Listen for incoming connections (backlog of 100)
    print("[+] Listening for connection...")
    client, addr = sock.accept()  # Accept a connection from a client
except Exception as e:
    print(f"[-] Listen failed: {e}")
    sys.exit(1)

print("[+] Got a connection!")

try:
    # Create a Paramiko Transport object for the SSH session
    bh_session = paramiko.Transport(client)
    bh_session.add_server_key(host_key)  # Add the server's host key
    server = Server()  # Create an instance of the custom SSH server class

    try:
        bh_session.start_server(server=server)  # Start the SSH server
    except paramiko.SSHException:
        print("[-] SSH Negotiation failed")

    # Accept an SSH channel (timeout after 20 seconds)
    chan = bh_session.accept(20)
    print("[+] Authenticated!")
    print(chan.recv(1024))  # Receive data from the client (up to 1024 bytes)
    chan.send("Welcome to bhp_ssh!")  # Send a welcome message to the client

    # Main loop for handling commands
    while True:
        try:
            command = input("Enter command: ").strip("\n")  # Get a command from the user
            if command != "exit":
                chan.send(command)  # Send the command to the client
                print(chan.recv(1024).decode(errors="ignore") + "\n")  # Print the command output
            else:
                chan.send("exit")  # Send "exit" to the client
                print("Exiting...")
                bh_session.close()  # Close the SSH session
                raise Exception("exit")  # Exit the loop
        except KeyboardInterrupt:
            bh_session.close()  # Handle Ctrl+C gracefully
        except Exception as e:
            print(f"[-] Caught exception: {str(e)}")
            bh_session.close()
finally:
    sys.exit(1)  # Exit the program
```

---

### How the Code Works

1. **Import Required Modules**:
   - `socket`: For creating and managing network sockets.
   - `paramiko`: For implementing SSH functionality.
   - `threading`: For thread synchronization in the `Server` class.
   - `sys`: For handling command-line arguments and system-specific functions.

2. **Load the Server's Host Key**:
   - The server's RSA key (`test_rsa.key`) is loaded for SSH authentication.

3. **Define the SSH Server Class**:
   - The `Server` class inherits from `paramiko.ServerInterface` and implements methods for handling channel requests and password authentication.

4. **Set Up the Socket**:
   - A TCP socket is created and bound to the specified IP address and port.
   - The server listens for incoming connections and accepts the first client.

5. **Start the SSH Server**:
   - A `paramiko.Transport` object is created for the SSH session.
   - The server's host key is added, and the SSH server is started.

6. **Handle Client Commands**:
   - The server enters a loop where it waits for commands from the user.
   - Commands are sent to the client, and their output is displayed.
   - The loop continues until the user enters "exit" or an exception occurs.

7. **Graceful Shutdown**:
   - The SSH session is closed, and the program exits gracefully.

---

### How to Run the Code

1. **Install Dependencies**:
   - Install the `paramiko` library using pip:
     ```bash
     pip install paramiko
     ```

2. **Generate an RSA Key**:
   - Generate an RSA key for the server using the following command:
     ```bash
     ssh-keygen -t rsa -f test_rsa.key
     ```
   - This will create two files: `test_rsa.key` (private key) and `test_rsa.key.pub` (public key).

3. **Run the Script**:
   - Save the script to a file, e.g., `ssh_server.py`.
   - Run the script with the server IP and port as arguments:
     ```bash
     python ssh_server.py 0.0.0.0 2222
     ```
   - Replace `0.0.0.0` with your server's IP address and `2222` with the desired port.

4. **Connect to the SSH Server**:
   - Use an SSH client (e.g., OpenSSH) to connect to the server:
     ```bash
     ssh root@<server_ip> -p 2222
     ```
   - Use the password `toor` for authentication.

5. **Execute Commands**:
   - Enter commands in the server's terminal, and they will be executed on the client.

---

### Notes
- This script is for educational purposes and should not be used in production environments.
- The server uses hardcoded credentials (`root:toor`), which is insecure. In a real-world scenario, use secure authentication methods.
- Error handling is minimal. Add more robust error handling for production use.
- Ensure the server's RSA key is kept secure and not shared publicly.
