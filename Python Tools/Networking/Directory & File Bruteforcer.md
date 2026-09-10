# **🔎 Directory & File Bruteforcer**

This script **brute-forces directories and files** on a target website using a **wordlist** and optional **file extensions**. It runs **multi-threaded**, making it **fast** and **efficient** for penetration testing.


- [cain.txt](https://github.com/aw-junaid/Black-Hat-Python/blob/main/Python%20Tools/Extras/cain.txt)
- [SVN_all.txt](https://github.com/aw-junaid/Black-Hat-Python/blob/main/Python%20Tools/Extras/SVN_all.txt)

```python
import queue
import threading
import urllib.error
import urllib.parse
import urllib.request

threads = 50
target_url = "http://testphp.vulnweb.com"
wordlist_file = "all.txt"
file_extensions = [".php", ".bak", ".orig", ".inc"]
resume = None
user_agent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36"

def build_wordlist(word_list_file):
    with open (word_list_file, "r") as l:
        raw_words = [line.rstrip("\n") for line in l]

    found_resume = False
    words = queue.Queue()

    for word in raw_words:
        if resume:
            if found_resume:
                words.put(word)
            else:
                if word == resume:
                    found_resume = True
                    print(f"Resuming wordlist from: {resume}")
        else:
            words.put(word)
    return words

def dir_bruter(extensions=None):
    while not word_queue.empty():
        attempt = word_queue.get()
        attempt_list = []

        # check if there's a file extensions
        # if not it's a directory path
        if "." not in attempt:
            attempt_list.append(f"/{attempt}/")
        else:
            attempt_list.append(f"/{attempt}")
        
        # if we want to bruteforce extensions
        if extensions:
            for extension in extensions:
                attempt_list.append(f"/{attempt}{extension}")
            
        for brute in attempt_list:
            url = f"{target_url}{urllib.parse.quote(brute)}"
            try:
                headers = {"User-Agent": user_agent}
                r = urllib.request.Request(url, headers=headers)
                response = urllib.request.urlopen(r)
                if len(response.read()):
                    print(f"[{response.code}] => {url}")
            except urllib.error.HTTPError as e:
                if e.code != 404:
                    print(f"!!! {e.code} => {url}")

word_queue = build_wordlist(wordlist_file)

for thread in range(threads):
    t = threading.Thread(target=dir_bruter, args=(file_extensions,))
    t.start()
```


## **🔹 Features**
✔ **Multi-threaded scanning** (default: 50 threads)  
✔ Supports **custom wordlists** (`all.txt`)  
✔ Can **resume** from a specific word in case of interruptions  
✔ Attempts to discover **hidden directories & files**  
✔ Adds **file extensions** (`.php, .bak, .orig, .inc`) to brute-force filenames  
✔ Uses a **custom User-Agent** to evade detection  

---

## **📌 Code Breakdown**  

### **1️⃣ Import Required Modules**
```python
import queue
import threading
import urllib.error
import urllib.parse
import urllib.request
```
- **`queue`** → Stores words from the wordlist for multi-threaded processing  
- **`threading`** → Runs multiple scanning threads concurrently  
- **`urllib`** → Sends HTTP requests and handles URL encoding  

---

### **2️⃣ Configuration Variables**
```python
threads = 50  # Number of concurrent threads
target_url = "http://testphp.vulnweb.com"  # Target website
wordlist_file = "all.txt"  # Path to wordlist file
file_extensions = [".php", ".bak", ".orig", ".inc"]  # Extensions to test
resume = None  # Resume scanning from a specific word
user_agent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36"
```
- **`threads`** → Determines how many **parallel** scans run (default: 50)  
- **`target_url`** → The website being tested  
- **`wordlist_file`** → The file containing directory & file names  
- **`file_extensions`** → Appends extensions to filenames (e.g., `admin.php`)  
- **`resume`** → If set, resumes scanning from a **specific word** in the list  
- **`user_agent`** → Mimics a browser request to **avoid detection**  

---

### **3️⃣ Load Wordlist into Queue**
```python
def build_wordlist(word_list_file):
    with open(word_list_file, "r") as l:
        raw_words = [line.rstrip("\n") for line in l]

    found_resume = False
    words = queue.Queue()

    for word in raw_words:
        if resume:
            if found_resume:
                words.put(word)
            else:
                if word == resume:
                    found_resume = True
                    print(f"Resuming wordlist from: {resume}")
        else:
            words.put(word)
    return words
```
- **Reads wordlist (`all.txt`)** and removes newline characters  
- **Stores words in a queue** (`word_queue`) for **multi-threading**  
- **Supports resuming** if the script was stopped midway  

---

### **4️⃣ Brute-force Directory & File Discovery**
```python
def dir_bruter(extensions=None):
    while not word_queue.empty():
        attempt = word_queue.get()
        attempt_list = []

        # Check if it's a directory (no dot in name)
        if "." not in attempt:
            attempt_list.append(f"/{attempt}/")
        else:
            attempt_list.append(f"/{attempt}")
        
        # Append file extensions to the word
        if extensions:
            for extension in extensions:
                attempt_list.append(f"/{attempt}{extension}")
            
        for brute in attempt_list:
            url = f"{target_url}{urllib.parse.quote(brute)}"
            try:
                headers = {"User-Agent": user_agent}
                r = urllib.request.Request(url, headers=headers)
                response = urllib.request.urlopen(r)
                
                if len(response.read()):  # If response contains data
                    print(f"[{response.code}] => {url}")
            except urllib.error.HTTPError as e:
                if e.code != 404:  # Ignore "Not Found" errors
                    print(f"!!! {e.code} => {url}")
```
- If **no dot (`.`)** in word → Treat as **directory** (`/admin/`)  
- If **dot (`.`)** → Treat as **file** (`config.php`)  
- Adds **file extensions** to filenames (e.g., `index.php`)  
- Sends an HTTP **request** with a **browser-like User-Agent**  
- Prints **valid responses** (e.g., `200 OK`, `403 Forbidden`)  
- Ignores `404 Not Found` errors  

---

### **5️⃣ Start Multi-threaded Brute-forcing**
```python
word_queue = build_wordlist(wordlist_file)

for thread in range(threads):
    t = threading.Thread(target=dir_bruter, args=(file_extensions,))
    t.start()
```
- Loads **wordlist** into `word_queue`  
- Spawns **50 concurrent threads** (by default)  
- Each thread runs `dir_bruter()`, testing different paths  

---

## **🚀 How to Run the Script**
### **📌 Prerequisites**
Ensure you have **Python 3.x** installed and a valid **wordlist** (`all.txt`).

### **📌 Update Configuration**
- Set `target_url` to the **website** you want to scan  
- Update `wordlist_file` to your **wordlist path**  
- Modify `file_extensions` if needed  

### **📌 Run the Script**
```bash
python script.py
```

---

## **🎯 Example Output**
```bash
[200] => http://testphp.vulnweb.com/admin/
[200] => http://testphp.vulnweb.com/config.php
[403] => http://testphp.vulnweb.com/secret/
[500] => http://testphp.vulnweb.com/debug.bak
```
- **`200 OK`** → File/Directory **exists** ✅  
- **`403 Forbidden`** → Exists but **restricted** 🔒  
- **`500 Internal Server Error`** → Might be **misconfigured** ⚠  
- **`404 Not Found`** → **Ignored** (doesn’t exist) ❌  

---

## **🎭 Summary**
✔ **Finds hidden directories & files** on websites  
✔ Uses **multi-threading** for faster scanning  
✔ **Ignores 404 errors** to reduce noise  
✔ **Can resume** scanning from a specific word  
✔ **Custom User-Agent** avoids detection  

This tool is useful for **ethical hacking, bug bounty hunting, & security audits** to **find unprotected files & directories**. 🚀  

---
