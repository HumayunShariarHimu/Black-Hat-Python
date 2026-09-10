This Python script demonstrates how to create, list, and delete a file using the `subprocess` module to execute shell commands.

---

### **Breakdown of the Code**
```python
import subprocess  # Import the subprocess module

# Create a new empty file named 'sample.txt'
subprocess.call(["touch", "sample.txt"])

# List all files in the current directory
subprocess.call(["ls"])

# Print confirmation message
print("Sample file created")

# Delete the file 'sample.txt'
subprocess.call(["rm", "sample.txt"])

# List files again to show that 'sample.txt' is deleted
subprocess.call(["ls"])

# Print confirmation message
print("Sample file deleted")
```

---

### **How It Works**
1. **`subprocess.call(["touch", "sample.txt"])`** → Runs the `touch sample.txt` command to create an empty file.
2. **`subprocess.call(["ls"])`** → Lists the contents of the current directory.
3. **`print("Sample file created")`** → Displays a confirmation message.
4. **`subprocess.call(["rm", "sample.txt"])`** → Runs `rm sample.txt` to delete the file.
5. **`subprocess.call(["ls"])`** → Lists files again, showing that `sample.txt` is no longer present.
6. **`print("Sample file deleted")`** → Displays another confirmation message.

---

### **Example Output**
```
file1.txt  file2.py  sample.txt
Sample file created
file1.txt  file2.py
Sample file deleted
```
- Initially, `sample.txt` is created and shown in the `ls` output.
- After deletion, `ls` no longer lists `sample.txt`.

---

### **How to Run the Script**
1. Save the script as `file_operations.py`.
2. Open a terminal and navigate to the script’s directory.
3. Run:
   ```sh
   python file_operations.py
   ```

---

### **Windows Compatibility**
- `touch` and `rm` are **not available** in Windows Command Prompt (`cmd`).
- Use **PowerShell equivalents**:
  ```python
  import subprocess
  
  # Create a file
  subprocess.call(["powershell", "New-Item", "-Path", "sample.txt", "-ItemType", "File"])
  
  # List files
  subprocess.call(["dir"], shell=True)
  
  print("Sample file created")
  
  # Delete the file
  subprocess.call(["powershell", "Remove-Item", "sample.txt"])
  
  # List files again
  subprocess.call(["dir"], shell=True)
  
  print("Sample file deleted")
  ```

---

### **Improved Version with Error Handling**
To handle potential errors, modify the script as follows:
```python
import subprocess

try:
    subprocess.check_call(["touch", "sample.txt"])
    subprocess.check_call(["ls"])
    print("Sample file created")

    subprocess.check_call(["rm", "sample.txt"])
    subprocess.check_call(["ls"])
    print("Sample file deleted")
except subprocess.CalledProcessError as e:
    print("Error:", e)
```
- `subprocess.check_call()` raises an exception if a command fails.
- Ensures error handling in case of permission issues or missing commands.
