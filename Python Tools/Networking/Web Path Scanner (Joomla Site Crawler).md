# **Web Path Scanner (Joomla Site Crawler)**  

This script is a **multi-threaded web scanner** that attempts to discover **hidden files & directories** on a target website by comparing it with a **local Joomla copy**.


- [cain.txt](https://github.com/aw-junaid/Black-Hat-Python/blob/main/Python%20Tools/Extras/cain.txt)
- [SVN_all.txt](https://github.com/aw-junaid/Black-Hat-Python/blob/main/Python%20Tools/Extras/SVN_all.txt)



```python
import os
import queue
import threading
import urllib.error
import urllib.parse
import urllib.request

threads = 10
target = "http://www.test.com"

# here you should download your local copy of Joomla for testing purposes
directory = "/Users/user/Downloads/joomla-x.x.x"  

# filtering out the file extensions we don't need. Feel free to extend! 
filters = [".jpg", ".gif", ".png", ".css"]

os.chdir(directory)
web_paths = queue.Queue()

for r, d, f in os.walk("."):
    for files in f:
        remote_path = f"{r}/{files}"
        if remote_path.startswith("."):
            remote_path = remote_path[1:]
        if os.path.splitext(files)[1] not in filters:
            web_paths.put(remote_path)

def test_remote():
    while not web_paths.empty():
        path = web_paths.get()
        url = f"{target}{path}"
        request = urllib.request.Request(url)
        try:
            response = urllib.request.urlopen(request)
            print(f"[{response.code}] => {path}")
            response.close()
        except urllib.error.HTTPError as e:
            print(f"Failed for error {e}, {e.code}") 

for thread in range(threads):
    print(f"Spawning thread: {thread}")
    t = threading.Thread(target=test_remote)
    t.start() 
```

# **Web Path Scanner (Joomla Site Crawler)**  

This script is a **multi-threaded web scanner** that attempts to discover **hidden files & directories** on a target website by comparing it with a **local Joomla copy**.

---

## **🔹 Features**
✔ Scans a Joomla site by checking if files exist remotely  
✔ Uses **multi-threading** for faster scanning (default: 10 threads)  
✔ Ignores unnecessary file types (`.jpg, .gif, .png, .css`)  
✔ Handles **HTTP errors gracefully**  

---

## **📌 Code Breakdown**  

### **1️⃣ Import Required Modules**
```python
import os
import queue
import threading
import urllib.error
import urllib.parse
import urllib.request
```
- **os** → Used for navigating local directories  
- **queue** → Stores file paths for multi-threaded scanning  
- **threading** → Runs multiple scans concurrently  
- **urllib** → Sends requests to the target website  

---

### **2️⃣ Define Configuration Variables**
```python
threads = 10  # Number of concurrent threads
target = "http://www.test.com"  # Target website URL

directory = "/Users/user/Downloads/joomla-x.x.x"  # Path to local Joomla copy

filters = [".jpg", ".gif", ".png", ".css"]  # File types to ignore
```
- **`threads`** → The number of threads used for parallel requests  
- **`target`** → The website to scan  
- **`directory`** → Local Joomla installation (update as needed)  
- **`filters`** → Excludes unnecessary file types (e.g., images & stylesheets)  

---

### **3️⃣ Populate the Queue with Local Paths**
```python
os.chdir(directory)  # Change to Joomla directory
web_paths = queue.Queue()

for r, d, f in os.walk("."):
    for files in f:
        remote_path = f"{r}/{files}"
        if remote_path.startswith("."):
            remote_path = remote_path[1:]
        if os.path.splitext(files)[1] not in filters:
            web_paths.put(remote_path)
```
- **`os.walk(directory)`** → Recursively scans all files  
- **Excludes** hidden paths (starting with `"."`)  
- **Ignores** filtered file types  
- **Stores valid paths** in a `queue.Queue()` for processing  

---

### **4️⃣ Function to Test Remote Files**
```python
def test_remote():
    while not web_paths.empty():
        path = web_paths.get()
        url = f"{target}{path}"  # Construct full URL
        request = urllib.request.Request(url)

        try:
            response = urllib.request.urlopen(request)
            print(f"[{response.code}] => {path}")  # Print status
            response.close()
        except urllib.error.HTTPError as e:
            print(f"Failed for error {e}, {e.code}")  # Handle HTTP errors
```
- **Constructs URLs** using the `target` and discovered `path`  
- **Sends HTTP requests** to check if the file exists  
- **Prints HTTP status codes** (e.g., `200 OK`, `404 Not Found`)  
- **Handles errors** gracefully (e.g., `403 Forbidden`, `404 Not Found`)  

---

### **5️⃣ Launch Multi-threaded Scanner**
```python
for thread in range(threads):
    print(f"Spawning thread: {thread}")
    t = threading.Thread(target=test_remote)
    t.start()
```
- **Starts `10` threads** (default) for parallel scanning  
- **Each thread runs `test_remote()`**, reducing scan time  

---

## **🚀 How to Run the Script**
### **📌 Prerequisites**
Ensure you have **Python 3.x** installed.

### **📌 Update Configuration**
- Set `target` to the website you want to scan  
- Update `directory` with the path to your **local Joomla copy**  

### **📌 Run the Script**
```bash
python script.py
```

---

## **🎯 Example Output**
```bash
Spawning thread: 0
Spawning thread: 1
Spawning thread: 2
...
[200] => /index.php
[200] => /administrator/index.php
[403] => /config.php
[404] => /secret-folder/
```
- `200 OK` → File exists on the target site ✅  
- `403 Forbidden` → File exists but is **restricted** 🔒  
- `404 Not Found` → File **does not exist** ❌  

---

## **🎭 Summary**
✔ Uses **local Joomla files** to find hidden web paths  
✔ **Multi-threaded** for fast scanning  
✔ Handles **HTTP errors** (403, 404, etc.)  
✔ Ignores **unimportant files**  

This script is useful for **penetration testing** & **security audits** to detect **unprotected files** on a website.  

