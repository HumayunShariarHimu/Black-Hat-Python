This script is designed to extract password hashes from a Windows memory dump using Volatility 3, a popular memory forensics tool. Specifically, it targets the **SAM** (Security Account Manager) and **System** registry hives within the memory dump to extract password hashes.


```python
import sys
import volatility.conf as conf
import volatility.registry as registry
import volatility.commands as commands
import volatility.addrspace as addrspace
from volatility.plugins.registry.registryapi import RegistryApi
from volatility.plugins.registry.lsadump import HashDump

memory_file = "Win10X64.vmem" # see README.md on how to get memory dumps

sys.path.append("/volatility3.1.0.0")

registry.PluginImporter()
config = conf.ConfObject()

config.parse_options()
config.PROFILE = "Win10X64"
config.LOCATION = f"file://{memory_file}"

registry.register_global_options(config, commands.Command)
registry.register_global_options(config, addrspace.BaseAddressSpace)

registry = RegistryApi(config)
registry.populate_offsets()

sam_offset = None
sys_offset = None

for offset in registry.all_offsets:
    
    if registry.all_offsets[offset].endswith("\\SAM"):
        sam_offset = offset
        print("[*] System: 0x%08x" % offset)
        
    if sam_offset is not None and sys_offset is not None:
        config.sys_offset = sys_offset
        config.sam_offset = sam_offset
        hashdump = HashDump(config)
        
        for hash in hashdump.calculate():
            print(hash)
        break
    
if sam_offset is None or sys_offset is None:
    print("[*] Failed to find the system or SAM offsets")
```

### Key Components:

1. **Memory File**:
   - **`memory_file = "Win10X64.vmem"`**: This represents the path to the memory dump file (e.g., `Win10X64.vmem`), which is the memory image from a Windows machine. You would need a memory dump file to run this script.

2. **Volatility 3 Imports**:
   - `volatility.conf`, `volatility.registry`, `volatility.commands`, `volatility.addrspace`, etc., are all imports from the Volatility 3 framework.
   - These modules help in managing configuration options, interacting with the memory address space, and loading the appropriate plugins for registry analysis and hash dumping.

3. **Registry and SAM/Hive Parsing**:
   - The **RegistryApi** is used to interact with the Windows registry hives that are loaded from the memory image.
   - The **`HashDump`** plugin is used to dump password hashes from the loaded registry hives (`SAM` and `System`).
   - The script searches for specific offsets in the memory dump that correspond to the **SAM** and **System** registry hives.

4. **Configuration Setup**:
   - The `config` object is created and configured with the path to the memory file and the profile (`Win10X64`) to match the Windows version of the memory dump.
   - **`registry.register_global_options()`**: Registers configuration options globally for the analysis (address space, commands, etc.).

5. **Searching for SAM and System Offsets**:
   - The script loops through the available offsets in the registry and looks for the `SAM` and `System` registry hives.
   - If both hives are found (`sam_offset` and `sys_offset`), the script proceeds to dump the password hashes.

6. **Hash Dump**:
   - If both the `SAM` and `System` offsets are found, it uses the **`HashDump`** plugin to dump the password hashes from these hives.
   - It prints the hashes in the format that Volatility outputs, which usually includes the usernames and their corresponding password hashes.

### Workflow:
1. **Initialize Config**: The configuration object is set with the memory dump file and profile.
2. **Search Offsets**: The script searches through the registry offsets in the memory dump to find the locations for `SAM` and `System`.
3. **Hash Dump**: Once both registry hives are located, it uses Volatility’s `HashDump` plugin to extract and print the password hashes from the memory dump.

### How to Run:

1. **Install Volatility 3**:
   - First, ensure you have Volatility 3 installed. You can clone the repository from GitHub:
     ```bash
     git clone https://github.com/volatilityfoundation/volatility3.git
     cd volatility3
     pip install .
     ```

2. **Set Up Memory Dump**:
   - Obtain a memory dump from the target machine (this can be done using tools like FTK Imager or WinPMEM). Save the memory dump (e.g., `Win10X64.vmem`) in the same directory as the script, or modify the `memory_file` variable to point to the correct location of the memory dump file.

3. **Install Dependencies**:
   - Install any required Python dependencies, such as `volatility` and `pywin32` (if needed for specific Volatility plugins):
     ```bash
     pip install volatility3 pywin32
     ```

4. **Run the Script**:
   - Execute the script to start parsing the memory dump and extracting the password hashes.
     ```bash
     python extract_hashes.py
     ```

5. **Results**:
   - The script will output the found hashes (if any) on the console, showing the usernames and the corresponding password hashes.

### Example Output:
If the `SAM` and `System` hives are found, the script will print something like:
```
[*] System: 0x12345678
[*] Found the SAM offset
user1:$HASH1
user2:$HASH2
...
```

### Important Considerations:

- **Memory Dump**: The success of this script depends on having a valid and complete memory dump file from the target system.
- **Legal and Ethical Use**: This script should only be used in controlled, ethical environments, such as penetration testing engagements where you have explicit authorization. Using this script without proper consent is illegal and unethical.
- **Volatility Version**: Make sure you're using **Volatility 3** (not Volatility 2) because the script imports from Volatility 3 libraries and plugins.
