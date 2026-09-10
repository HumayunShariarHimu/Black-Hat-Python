This Python script lists and sorts the contents of a specified directory using the `os` module.

---

### **Breakdown of the Code**
```python
import os  # Import the OS module for interacting with the file system
import sys  # Import the sys module for command-line arguments

# List directory contents and sort them alphabetically
print(sorted(os.listdir(sys.argv[1])))
```

---

### **How It Works**
1. **`sys.argv[1]`** → Gets the directory path from the command-line argument.
2. **`os.listdir(directory)`** → Lists all files and folders in the specified directory.
3. **`sorted(...)`** → Sorts the list alphabetically.
4. **`print(...)`** → Displays the sorted list.

---

### **How to Run the Script**
1. **Save the script** as `list_dir.py`.
2. **Open a terminal** and run:
   ```sh
   python list_dir.py /path/to/directory
   ```
   - Replace `/path/to/directory` with the actual directory you want to list.
   - Example for Linux/macOS:
     ```sh
     python list_dir.py /home/user/Documents
     ```
   - Example for Windows:
     ```sh
     python list_dir.py C:\Users\User\Documents
     ```

---

### **Example Output**
If the `/home/user/Documents` directory contains:
```
report.docx
notes.txt
projects
assignments
```
Running the script:
```sh
python list_dir.py /home/user/Documents
```
Would output:
```python
['assignments', 'notes.txt', 'projects', 'report.docx']
```
- The list is **sorted alphabetically**.

---

### **Handling Errors Gracefully**
If the user forgets to provide an argument or provides an invalid directory, the script should handle it:

```python
import os
import sys

# Ensure the user provides a directory path
if len(sys.argv) < 2:
    print("Usage: python list_dir.py <directory_path>")
    sys.exit(1)

directory = sys.argv[1]

# Check if the directory exists
if not os.path.isdir(directory):
    print(f"Error: '{directory}' is not a valid directory.")
    sys.exit(1)

# List and print the sorted directory contents
print(sorted(os.listdir(directory)))
```
**Enhancements:**
✅ Checks if a directory path is provided.  
✅ Validates if the directory exists.  
✅ Displays an error message if the path is incorrect.  

---

### **Cross-Platform Compatibility**
- Works on **Windows, Linux, and macOS**.
- Uses `os.listdir()`, which is available on all platforms.

---

