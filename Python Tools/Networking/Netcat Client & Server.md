# Netcat replacement

```python
import sys
import socket
import getopt
import threading
import subprocess

# Global variables to store options
listen = False
command = False
upload = False
execute = ""
target = ""
upload_destination = ""
port = 0

def run_command(cmd):
    """Executes a command on the system and returns the output."""
    cmd = cmd.rstrip()
    
    try:
        output = subprocess.check_output(cmd, stderr=subprocess.STDOUT, shell=True)
    except subprocess.CalledProcessError as e:
        output = e.output
    
    return output


def client_handler(client_socket):
    """Handles incoming client connections."""
    global upload
    global execute
    global command

    # Check if an upload destination is specified
    if len(upload_destination):
        file_buffer = b""
        
        while True:
            data = client_socket.recv(1024)
            if not data:
                break
            else:
                file_buffer += data
        
        try:
            with open(upload_destination, "wb") as file_descriptor:
                file_descriptor.write(file_buffer)
            client_socket.send(f"Successfully saved file to {upload_destination}".encode())
        except OSError as e:
            client_socket.send(f"Failed to save file to {upload_destination} due to OS Error. Details: {e}".encode())
    
    # Check if a command should be executed
    if len(execute):
        output = run_command(execute)
        client_socket.send(output)
    
    # If command shell mode is enabled
    if command:
        while True:
            client_socket.send(b"<BHP:#> ")
            cmd_buffer = b""
            while b"\n" not in cmd_buffer:
                cmd_buffer += client_socket.recv(1024)
            response = run_command(cmd_buffer.decode())
            client_socket.send(response)


def server_loop():
    """Starts a server and listens for incoming connections."""
    global target
    global port
    
    if not len(target):
        target = "0.0.0.0"  # Listen on all available interfaces
    
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind((target, port))
    server.listen(5)
    
    while True:
        client_socket, addr = server.accept()
        client_thread = threading.Thread(target=client_handler, args=(client_socket,))
        client_thread.start()


def client_sender(buffer):
    """Sends data to a remote server as a client."""
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
    try:
        client.connect((target, port))
        
        if len(buffer):
            client.send(buffer.encode())
        
        while True:
            recv_len = 1
            response = b""
            
            while recv_len:
                data = client.recv(4096)
                recv_len = len(data)
                response += data
                if recv_len < 4096:
                    break
            
            print(response.decode(), end=" ")
            buffer = input("") + "\n"
            client.send(buffer.encode())
    
    except socket.error as e:
        print("[*] Exception caught. Exiting.")
        print(f"[*] Details of error: {e}")
        client.close()


def usage_info():
    """Displays usage information for the script."""
    print("Netcat Replacement")
    print("Usage: bhp_net.py -t target_host -p port")
    print("-l --listen                  - Listen for incoming connections")
    print("-e --execute=file_to_run     - Execute a file upon receiving a connection")
    print("-c --command                 - Initialize a command shell")
    print("-u --upload=destination      - Upload a file and write it to [destination]")
    print("Examples:")
    print("bhp_net.py -t 192.168.0.1 -p 555 -l -c")
    print("bhp_net.py -t 192.168.0.1 -p 555 -l -u=c:\\target.exe")
    print("bhp_net.py -t 192.168.0.1 -p 555 -l -e=\"cat /etc/passwd\"")
    print("echo 'ABCDEFGHI' | ./bhp_net.py -t 192.168.11.12 -p 135")
    sys.exit()


def main():
    """Main function to parse arguments and start client/server operations."""
    global listen
    global port
    global execute
    global command
    global upload_destination
    global target

    if not len(sys.argv[1:]):
        usage_info()
    
    try:
        opts, args = getopt.getopt(sys.argv[1:], "hle:t:p:cu:", ["help", "listen", "execute=", "target=", "port=", "command", "upload="])
        for o, a in opts:
            if o in ("-h", "--help"):
                usage_info()
            elif o in ("-l", "--listen"):
                listen = True
            elif o in ("-e", "--execute"):
                execute = a
            elif o in ("-c", "--command"):
                command = True
            elif o in ("-u", "--upload"):
                upload_destination = a
            elif o in ("-t", "--target"):
                target = a
            elif o in ("-p", "--port"):
                port = int(a)
            else:
                assert False, "Unhandled option"
    
    except getopt.GetoptError as e:
        print(str(e))
        usage_info()
    
    if not listen and len(target) and port > 0:
        buffer = sys.stdin.read()
        client_sender(buffer)
    
    if listen:
        server_loop()

if __name__ == "__main__":
    main()
```

### Explanation:

- This script is a **Netcat replacement**, allowing you to send and receive data over a network.
- It can operate in **client mode** (sending data) or **server mode** (listening for connections).
- The script supports:
  - **Executing commands** on the remote machine.
  - **Uploading files** to the target machine.
  - **Creating an interactive shell** for remote access.

### How to Run:

#### 1. Start a Listener (Server Mode)
```bash
python bhp_net.py -t 0.0.0.0 -p 4444 -l -c
```
This sets up a listening shell on port 4444.

#### 2. Connect as a Client
```bash
python bhp_net.py -t 192.168.1.10 -p 4444
```
This connects to the listening server.

#### 3. Send a File
```bash
python bhp_net.py -t 192.168.1.10 -p 4444 -l -u=/tmp/shell.sh
```
This listens and writes the received file to `/tmp/shell.sh`.
