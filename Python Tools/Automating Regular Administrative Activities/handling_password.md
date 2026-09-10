This Python script establishes an SSH connection to a remote machine using `paramiko`, creates a directory on the remote system, and then closes the connection.

---

### **Breakdown of the Code**
```python
import sys
import paramiko
import time
```
- `sys`: Used for system-related functions (not used in this script but may be useful for error handling).
- `paramiko`: A Python library for SSH connections.
- `time`: Provides time-related functions (not used in this script, but useful for delays if needed).

---

### **Connecting to the Remote Machine**
```python
ip_address = "192.168.2.106"  # Remote machine IP address
username = "student"  # SSH username
password = "training"  # SSH password

ssh_client = paramiko.SSHClient()  # Initialize SSH client
ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())  # Automatically accept unknown host keys
ssh_client.load_system_host_keys()  # Load known SSH keys
```
- This sets up the SSH client and automatically adds the host key if it's not already known.

---

### **Authenticating and Establishing an SSH Connection**
```python
ssh_client.connect(hostname=ip_address, username=username, password=password)
print("Successful connection", ip_address)
```
- Connects to the remote server using the provided IP, username, and password.
- Prints a success message if the connection is established.

---

### **Executing Remote Commands**
```python
ssh_client.invoke_shell()  # Open an interactive shell session
remote_connection = ssh_client.exec_command('cd Desktop; mkdir work\n')
remote_connection = ssh_client.exec_command('mkdir test_folder\n')
```
- `exec_command()` runs commands on the remote system.
- `cd Desktop; mkdir work` → Moves to the `Desktop` and creates a `work` directory.
- `mkdir test_folder` → Creates another folder in the current directory.

---

### **Closing the SSH Connection**
```python
ssh_client.close
```
- **Incorrect usage**: `ssh_client.close` should be `ssh_client.close()`.
- **Correct version**:
  ```python
  ssh_client.close()
  ```
- This properly terminates the SSH session.

---

### **Expected Output**
If successful:
```
Successful connection 192.168.2.106
```
- The folders **work** (inside `Desktop`) and **test_folder** (inside the home directory) are created on the remote system.

---

### **How to Run the Script**
1. **Install Paramiko** (if not installed):
   ```sh
   pip install paramiko
   ```
2. **Save the script** as `ssh_remote.py`.
3. **Run it**:
   ```sh
   python ssh_remote.py
   ```

---

### **Improved Version with Error Handling**
To handle potential errors, modify the script:
```python
import paramiko

ip_address = "192.168.2.106"
username = "student"
password = "training"

try:
    ssh_client = paramiko.SSHClient()
    ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh_client.load_system_host_keys()
    ssh_client.connect(hostname=ip_address, username=username, password=password)
    print("Successful connection", ip_address)

    commands = ['cd Desktop; mkdir work', 'mkdir test_folder']
    for cmd in commands:
        stdin, stdout, stderr = ssh_client.exec_command(cmd)
        print(stdout.read().decode(), stderr.read().decode())  # Print command output/errors

except Exception as e:
    print("Error:", e)

finally:
    ssh_client.close()
    print("SSH connection closed")
```
- **Handles connection failures gracefully**.
- **Executes multiple commands in a loop**.
- **Prints command output or errors**.

---

### **Security Considerations**
- **Avoid hardcoding credentials** → Use environment variables or a secure credential store.
- **Use SSH keys instead of passwords** → More secure than password-based authentication.
- **Verify host key fingerprint** → Prevents man-in-the-middle attacks.
