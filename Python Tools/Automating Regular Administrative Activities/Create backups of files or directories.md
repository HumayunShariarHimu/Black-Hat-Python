This Python script **performs a backup of a directory** using the `rsync` command and provides additional features like directory existence checks, confirmation prompts, and file deletion.

```python
import os
import shutil
import time
from sh import rsync
def check_dir(os_dir):
	if not os.path.exists(os_dir):
		print (os_dir, "does not exist.")
		exit(1)
def ask_for_confirm():
	ans = input("Do you want to Continue? yes/no\n")
	global con_exit
	if ans == 'yes':
		con_exit = 0
		return con_exit
	elif ans == "no":
		con_exit = 1
		return con_exit
	else:
		print ("Answer with yes or no.")
		ask_for_confirm()
def delete_files(ending):
	for r, d, f in os.walk(backup_dir):
		for files in f:
			if files.endswith("." + ending):
				os.remove(os.path.join(r, files))
backup_dir = input("Enter directory to backup\n")	# Enter directory name
check_dir(backup_dir)
print (backup_dir, "saved.")
time.sleep(3)
backup_to_dir= input("Where to backup?\n")
check_dir(backup_to_dir)
print ("Doing the backup now!")
ask_for_confirm()
if con_exit == 1:
	print ("Aborting the backup process!")
	exit(1)
rsync("-auhv", "--delete", "--exclude=lost+found", "--exclude=/sys", "--exclude=/tmp", "--exclude=/proc",
  "--exclude=/mnt", "--exclude=/dev", "--exclude=/backup", backup_dir, backup_to_dir)
```

---

## **Breakdown of the Code**
### **1. Import Required Modules**
```python
import os
import shutil
import time
from sh import rsync
```
- `os` → **File and directory operations**.  
- `shutil` → Not used in this script but generally used for file operations.  
- `time` → Used for delays (`time.sleep(3)`).  
- `sh` → Runs shell commands directly from Python (`rsync`).

---

### **2. Function to Check if Directory Exists**
```python
def check_dir(os_dir):
	if not os.path.exists(os_dir):
		print (os_dir, "does not exist.")
		exit(1)
```
- Ensures the given directory **exists** before proceeding.
- If the directory does **not exist**, the program **exits** with an error message.

---

### **3. Function to Ask for Confirmation**
```python
def ask_for_confirm():
	ans = input("Do you want to Continue? yes/no\n")
	global con_exit
	if ans == 'yes':
		con_exit = 0
		return con_exit
	elif ans == "no":
		con_exit = 1
		return con_exit
	else:
		print ("Answer with yes or no.")
		ask_for_confirm()
```
- **Prompts the user for confirmation** before proceeding.
- If the user inputs `"yes"`, backup proceeds.
- If `"no"`, the process **aborts**.
- If an invalid response is given, it **asks again recursively**.

---

### **4. Function to Delete Files of a Specific Type**
```python
def delete_files(ending):
	for r, d, f in os.walk(backup_dir):
		for files in f:
			if files.endswith("." + ending):
				os.remove(os.path.join(r, files))
```
- Recursively **deletes all files** with a specific extension inside `backup_dir`.

---

### **5. Get User Input for Directories**
```python
backup_dir = input("Enter directory to backup\n")	
check_dir(backup_dir)  # Validate directory
print (backup_dir, "saved.")
time.sleep(3)
```
- Asks the user to **enter the directory to back up**.
- **Checks if the directory exists** using `check_dir()`.
- **Pauses for 3 seconds** before proceeding.

```python
backup_to_dir= input("Where to backup?\n")
check_dir(backup_to_dir)
print ("Doing the backup now!")
```
- Asks for **destination directory**.
- **Checks if the directory exists** before proceeding.

---

### **6. Ask for Confirmation Before Backup**
```python
ask_for_confirm()
if con_exit == 1:
	print ("Aborting the backup process!")
	exit(1)
```
- **User confirms backup** (`yes` or `no`).
- If `"no"`, **exit the program**.

---

### **7. Perform the Backup Using `rsync`**
```python
rsync("-auhv", "--delete", "--exclude=lost+found", "--exclude=/sys", "--exclude=/tmp", "--exclude=/proc",
  "--exclude=/mnt", "--exclude=/dev", "--exclude=/backup", backup_dir, backup_to_dir)
```
- **Uses `rsync` to sync files between directories**:
  - `-a` → Archive mode (preserves permissions, timestamps, symbolic links, etc.).
  - `-u` → Skips files that are **newer in the destination**.
  - `-h` → Human-readable output.
  - `-v` → Verbose mode (shows progress).
  - `--delete` → Deletes files in **destination** that no longer exist in **source**.
  - `--exclude=...` → **Skips system directories** to prevent errors.

---

## **How to Run the Script**
1. **Ensure `sh` module is installed** (if not, install it):
   ```sh
   pip install sh
   ```
2. **Save the script as `backup_script.py`.**
3. **Run the script**:
   ```sh
   python backup_script.py
   ```
4. **Follow the prompts** to specify:
   - **Source directory** (to back up).
   - **Destination directory** (backup location).
   - **Confirm** if you want to proceed.

---

## **Example Scenario**
### ✅ **User Input**
```
Enter directory to backup
/home/user/documents
/home/user/documents saved.

Where to backup?
/home/user/backup
Doing the backup now!
Do you want to Continue? yes/no
yes
```
### ✅ **Expected Output**
```
sending incremental file list
documents/
documents/file1.txt
documents/file2.pdf

sent 1.23MB received 1.02KB 3.4MB/s
total size is 50MB speedup is 40.00x
```

---

## **Enhancements & Fixes**
### 🛠 **Fix Recursive Call in `ask_for_confirm()`**
```python
def ask_for_confirm():
	while True:
		ans = input("Do you want to Continue? yes/no\n")
		if ans.lower() == 'yes':
			return 0
		elif ans.lower() == "no":
			return 1
		else:
			print("Answer with yes or no.")
```
✅ **Avoids deep recursion**.  
✅ **Uses a loop instead of repeated function calls**.  

---

### 🔹 **Improving Error Handling**
Add exception handling to **catch errors gracefully**:
```python
try:
    rsync("-auhv", "--delete", backup_dir, backup_to_dir)
    print("Backup completed successfully!")
except Exception as e:
    print("Error during backup:", str(e))
```
✅ **Prevents crashes if `rsync` fails**.  
✅ **Displays a user-friendly error message**.  

---

## **Key Takeaways**
✅ **`rsync` efficiently syncs directories while preserving metadata**.  
✅ **`check_dir()` ensures valid directories are provided**.  
✅ **`ask_for_confirm()` prevents accidental backups**.  
✅ **`delete_files()` removes specific file types from `backup_dir`**.  
✅ **Handles missing directories and errors gracefully**.  

---
