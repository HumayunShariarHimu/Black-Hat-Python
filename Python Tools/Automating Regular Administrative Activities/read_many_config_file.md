This Python script **reads multiple INI configuration files** using the `configparser` module and **identifies which files were found and which were missing**.

```python
from configparser import ConfigParser
import glob
p = ConfigParser()
files = ['hello.ini', 'bye.ini', 'read_simple.ini', 'welcome.ini']
files_found = p.read(files)
files_missing = set(files) - set(files_found)
print('Files found:  ', sorted(files_found))
print('Files missing:  ', sorted(files_missing))
```


## **Breakdown of the Code**
### **1. Import Required Modules**
```python
from configparser import ConfigParser
import glob
```
- `ConfigParser` → Parses `.ini` configuration files.  
- `glob` → (Not used in this script but typically used for file pattern matching).

---

### **2. Initialize ConfigParser**
```python
p = ConfigParser()
```
- Creates a **configuration parser object**.

---

### **3. Define a List of INI Files to Read**
```python
files = ['hello.ini', 'bye.ini', 'read_simple.ini', 'welcome.ini']
```
- This **list contains the names** of INI files the script will attempt to read.

---

### **4. Read the Configuration Files**
```python
files_found = p.read(files)
```
- `p.read(files)` attempts to read the **existing files** from the given list.
- **Returns a list** of files that **exist and were successfully read**.

---

### **5. Identify Missing Files**
```python
files_missing = set(files) - set(files_found)
```
- Converts both lists into **sets** and subtracts them to determine **which files are missing**.

---

### **6. Print the Results**
```python
print('Files found:  ', sorted(files_found))
print('Files missing:  ', sorted(files_missing))
```
- Displays which files **were found and successfully read**.
- Lists **missing files**.

---

## **Example Scenario**
### ✅ **Existing Files in the Directory**
```
read_simple.ini
welcome.ini
```

### ✅ **Expected Output**
```
Files found:   ['read_simple.ini', 'welcome.ini']
Files missing:   ['bye.ini', 'hello.ini']
```

---

## **How to Run the Script**
1. **Create some INI files** in the same directory:
   - `read_simple.ini`
   - `welcome.ini`
   - (Leave `hello.ini` and `bye.ini` missing)
2. **Save the script** as `find_ini_files.py`.
3. **Run the script**:
   ```sh
   python find_ini_files.py
   ```

---

## **Enhancements: Checking for INI Files Dynamically**
Instead of manually listing files, **use `glob` to find all `.ini` files**:
```python
import glob
from configparser import ConfigParser

p = ConfigParser()

# Dynamically find all INI files in the current directory
files = glob.glob("*.ini")

files_found = p.read(files)
files_missing = set(files) - set(files_found)

print('Files found:  ', sorted(files_found))
print('Files unreadable (corrupt or empty):  ', sorted(files_missing))
```
✅ **Automatically detects all `.ini` files** in the directory.  
✅ **Handles unreadable or corrupt INI files**.

---

## **Key Takeaways**
✅ **`p.read(files)` loads multiple INI files at once**.  
✅ **Tracks missing files using `set()` operations**.  
✅ **Dynamically finds `.ini` files using `glob.glob("*.ini")`**.  
