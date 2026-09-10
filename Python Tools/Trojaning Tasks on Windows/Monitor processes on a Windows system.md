Monitor processes on a Windows system, capturing details about their creation, ownership, privileges, and other process-related information. It writes the captured information to a CSV log file:


```python
import win32con
import win32api
import win32security
import wmi
import os

LOG_FILE = "process_monitor_log.csv"

def get_process_privileges(pid):
    try:
        # obtain a handle to the target process
        hproc = win32api.OpenProcess(win32con.PROCESS_QUERY_INFORMATION,
                                     False,
                                     pid)
        
        # open main process token
        htok = win32security.OpenProcessToken(hproc, win32con.TOKEN_QUERY)
        
        # retrieve the list of privileges enabled
        privs = win32security.GetTokenInformation(htok, win32security.TokenPrivileges)
        
        # iterate over privileges and output the ones that are enabled
        priv_list = []
        for priv_id, priv_flags in privs:
            # check if privilege is enabled
            if priv_flags == 3:
                priv_list.append(win32security.LookupPrivilegeName(None, priv_id))
                
    except:
        priv_list.append("N/A")
        
    return "|".join(priv_list)

def log_to_file(message):
    with open(LOG_FILE, "ab") as fd:
        fd.write(f"{message}\r\n")
    return

# create a log file header
if not os.path.isfile(LOG_FILE):
    log_to_file("Time,User,Executable,CommandLine,PID,ParentPID,Privileges")
    
# instantiate the WMI interface
c = wmi.WMI()

# create our process monitor
process_watcher = c.Win32_Process.watch_for("creation")

while True:
    try:
        new_process = process_watcher()
        proc_owner = new_process.GetOwner()
        proc_owner = f"{proc_owner[0]}\\{proc_owner[2]}"
        create_date = new_process.CreationDate
        executable = new_process.ExecutablePath
        cmdline = new_process.CommandLine
        pid = new_process.ProcessId
        parent_pid = new_process.ParentProcessId
        
        privileges = get_process_privileges(pid)
        
        process_log_message = f"{create_date}, {proc_owner}, {executable}, {cmdline}, {pid}, {parent_pid}, {privileges}."
        
        print(f"{process_log_message}\r\n")
        
        log_to_file(process_log_message)
    except:
        pass
```


This code is designed to monitor processes on a Windows system, capturing details about their creation, ownership, privileges, and other process-related information. It writes the captured information to a CSV log file. Below is an explanation of each section and how to run the code:

### Breakdown:
1. **Imports:**
   - `win32con`: Contains constants used in the Win32 API, such as process access levels.
   - `win32api`: Provides functions for interacting with Windows APIs.
   - `win32security`: Allows interaction with security and permissions of processes.
   - `wmi`: Provides access to Windows Management Instrumentation (WMI) for querying system processes and events.
   - `os`: Used for checking the existence of the log file.

2. **Global Constants:**
   - `LOG_FILE`: Path to the CSV file where logs will be written. Default is `"process_monitor_log.csv"`.

3. **Function Definitions:**
   - **`get_process_privileges(pid)`**: 
     - Retrieves the privileges of a process given its `pid`.
     - Opens a process handle, retrieves its security token, and checks the privileges.
     - Returns a string of enabled privileges separated by a pipe (`|`), or "N/A" if no privileges can be retrieved.
   
   - **`log_to_file(message)`**: 
     - Writes the provided `message` to the log file in append mode.
   
4. **Log File Header:**
   - Before starting the monitoring, the script checks if the log file exists. If not, it creates the file and writes a header with the columns: `Time`, `User`, `Executable`, `CommandLine`, `PID`, `ParentPID`, and `Privileges`.

5. **WMI Interface and Process Monitoring:**
   - Uses the WMI interface to monitor processes on the system.
   - `process_watcher = c.Win32_Process.watch_for("creation")`: Listens for new processes being created.
   - In the loop, it retrieves the details of each newly created process, such as:
     - **Owner**: The user account that started the process.
     - **Creation Date**: The timestamp when the process was created.
     - **Executable Path**: The path of the executable that started the process.
     - **Command Line**: The full command used to launch the process.
     - **PID and Parent PID**: The process ID and the PID of its parent process.
     - **Privileges**: The process's privileges (using `get_process_privileges` function).

6. **Logging and Output:**
   - It prints and writes the details of each new process to the log file in CSV format. Each log entry contains the process's creation date, user, executable path, command line, PID, parent PID, and privileges.

### How to Run:

1. **Install Required Libraries:**
   - Install the required Python libraries:
     ```bash
     pip install pywin32 wmi
     ```

2. **Permissions:**
   - This script requires administrative privileges to query process privileges, so make sure to run it with elevated permissions.

3. **Run the Script:**
   - Save the code to a Python script (e.g., `process_monitor.py`).
   - Open a Command Prompt or PowerShell window with administrator rights.
   - Run the script:
     ```bash
     python process_monitor.py
     ```
   - It will start monitoring for new processes and log the relevant details to `process_monitor_log.csv`.

### Example Output (in CSV format):
```
Time,User,Executable,CommandLine,PID,ParentPID,Privileges
2025-02-06T12:34:56,DOMAIN\user,C:\Program Files\example\app.exe,"app.exe -flag",1234,5678,SeDebugPrivilege|SeBackupPrivilege
```

### Notes:
- The script continuously runs, watching for new processes being created. It will log all new processes' information as long as it is running.
- The CSV file will grow over time with logged process data.
