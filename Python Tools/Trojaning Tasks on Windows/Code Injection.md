# Code Injection

This script implements a Windows service that periodically executes a VBS script at regular intervals. It uses the `pywin32` library to interact with Windows services and manage the service lifecycle. 

- [bhservice_task.vbs.txt](https://github.com/aw-junaid/Black-Hat-Python/blob/main/Python%20Tools/Trojaning%20Tasks%20on%20Windows/bhservice_task.vbs.txt)

```python
import os
import servicemanager
import shutil
import subprocess
import sys

import win32event
import win32service
import win32serviceutil

SRCDIR = 'C:\\Users\\tim\\work'
TGTDIR = 'C:\\Windows\\TEMP'

class BHServerSvc(win32serviceutil.ServiceFramework):
    _svc_name_ = "BlackHatService"
    _svc_display_name_ = "Black Hat Service"
    _svc_description_ = ("Executes VBScripts at regular intervals." +
                        " What could possibly go wrong?")

    def __init__(self, args):
        self.vbs = os.path.join(TGTDIR, 'bhservice_task.vbs')
        self.timeout = 1000 * 60

        win32serviceutil.ServiceFramework.__init__(self, args)
        self.hWaitStop = win32event.CreateEvent(None, 0, 0, None)

    def SvcStop(self):
        self.ReportServiceStatus(win32service.SERVICE_STOP_PENDING)
        win32event.SetEvent(self.hWaitStop)

    def SvcDoRun(self):
        self.ReportServiceStatus(win32service.SERVICE_RUNNING)
        self.main()

    def main(self):
        while True:
            ret_code = win32event.WaitForSingleObject(self.hWaitStop, self.timeout)
            if ret_code == win32event.WAIT_OBJECT_0:
                servicemanager.LogInfoMsg("Service is stopping")
                break
            
            src = os.path.join(SRCDIR, 'bhservice_task.vbs')
            shutil.copy(src, self.vbs)
            subprocess.call("cscript.exe %s" % self.vbs, shell=False)
            os.unlink(self.vbs)

if __name__ == '__main__':
    if len(sys.argv) == 1:
        servicemanager.Initialize()
        servicemanager.PrepareToHostSingle(BHServerSvc)
        servicemanager.StartServiceCtrlDispatcher()
    else:
        win32serviceutil.HandleCommandLine(BHServerSvc)

```


This script implements a Windows service that periodically executes a VBS script at regular intervals. It uses the `pywin32` library to interact with Windows services and manage the service lifecycle. 

The script is structured as a Windows service, which performs the following tasks:

1. **Initialization**: It defines the service and its properties, including the service name, display name, and description.
2. **Service Control**: The service can be started and stopped using standard Windows service commands.
3. **Periodic Task Execution**: The service regularly copies a VBS file from a source directory to the Windows TEMP directory and executes it using `cscript.exe`.

### Key Components:

1. **Global Constants**:
   - `SRCDIR`: The source directory where the VBS script (`bhservice_task.vbs`) is located.
   - `TGTDIR`: The target directory where the VBS script is copied to (in this case, `C:\\Windows\\TEMP`).

2. **BHServerSvc Class**:
   - Inherits from `win32serviceutil.ServiceFramework`.
   - **`_svc_name_`**: The name of the service (`BlackHatService`).
   - **`_svc_display_name_`**: The display name for the service (`Black Hat Service`).
   - **`_svc_description_`**: A description of the service, which states it executes VBScripts at regular intervals.

3. **Service Lifecycle Management**:
   - **`__init__(self, args)`**: Initializes the service, including setting up the VBS file path and the timeout for the service's main loop.
   - **`SvcStop(self)`**: Stops the service by signaling that it is in a stop pending state.
   - **`SvcDoRun(self)`**: Runs the main service loop, starting the periodic execution of the VBS script.
   - **`main(self)`**: Contains the loop that handles the periodic execution. It:
     - Copies the VBS script from the source directory to the TEMP directory.
     - Executes the VBS script using `cscript.exe`.
     - Deletes the VBS script from the TEMP directory after execution.
     - Repeats the process every `timeout` (set to 60 seconds here).

4. **Service Control**:
   - The `if __name__ == '__main__':` block ensures the service is started and can be managed using Windows service commands like `start`, `stop`, `restart`.

### How It Works:
- The service runs in the background and executes the specified VBS file (`bhservice_task.vbs`) at regular intervals (every 60 seconds by default).
- The VBS file is copied from the source directory (`SRCDIR`) to the target directory (`TGTDIR`), executed, and then deleted.
- The `cscript.exe` utility is used to run the VBS script silently from the command line.

### How to Run the Script:

1. **Install Required Libraries**:
   - Make sure you have the `pywin32` library installed, as it provides the necessary modules to work with Windows services.
     ```bash
     pip install pywin32
     ```

2. **Setting Up the Service**:
   - Save the script as `bhserver_service.py` (or any preferred name).
   - Ensure the VBS script (`bhservice_task.vbs`) is located in the source directory (`C:\\Users\\tim\\work`).
   
3. **Running the Script**:
   - Open Command Prompt with **administrator rights**.
   - To install the service, run:
     ```bash
     python bhserver_service.py install
     ```
   - To start the service, run:
     ```bash
     python bhserver_service.py start
     ```

4. **Stopping the Service**:
   - To stop the service, run:
     ```bash
     python bhserver_service.py stop
     ```

5. **Uninstalling the Service**:
   - To remove the service, run:
     ```bash
     python bhserver_service.py remove
     ```

### Important Notes:
- **Security Warning**: The name "BlackHatService" and the functionality of executing potentially harmful scripts periodically suggests this script could be used for malicious purposes. It is strongly advised to not use this in any unauthorized or non-testing environments. It should only be used in a controlled, legal environment for educational or ethical penetration testing purposes.
- **Permissions**: The script requires elevated privileges (administrator rights) to install, start, stop, or uninstall Windows services.
- **VBS File**: Ensure the VBS file (`bhservice_task.vbs`) is safe and intended for testing or educational purposes.
