# **Burp Suite Extension: BHP Wordlist Generator**  

This Burp Suite extension **extracts words from HTTP responses** and **generates a password wordlist** for penetration testing. The wordlist includes **common words found on the target website**, making it useful for **password guessing attacks**.


```python
import re
from datetime import datetime
from html.parser import HTMLParser

from burp import IBurpExtender
from burp import IContextMenuFactory
from javax.swing import JMenuItem
from java.util import List, ArrayList
from java.net import URL


class TagStripper(HTMLParser):
    def __init__(self):
        HTMLParser.__init__(self)
        self.page_text = []

    def handleData(self, data):
        self.page_text.append(data)

    def handleComment(self, data):
        self.handle_data(data)

    def strup(self, html):
        self.feed(html)
        return " ".join(self.page_text)

class BurpExtender(IBurpExtender, IContextMenuFactory):
    def registerExtenderCallbacks(self, callbacks):
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        self.context = None
        self.hosts = set()

        # start with a common one
        self.wordlist = {"password"}

        callbacks.setExtensionName("BHP Wordlist")
        callbacks.registerContextMenuFactory(self)
        return
    
    def createMenuItems(self, context_menu):
        self.context = context_menu
        menu_list = ArrayList()
        menu_list.add(JMenuItem("Create Wordlist", actionPerformed=self.wordlist_menu))
        return menu_list

    def wordlistMenu(self, event):
        # grab the details of what the user clicked
        http_traffic = self.context.getSelectedMessages()

        for traffic in http_traffic:
            http_service = traffic.getHttpService()
            host = http_service.getHost()
            self.host.add(host)
            http_response = traffic.getResponse()
            if http_response:
                self.get_words(http_response)
        self.display_wordlist()
        return

    def getWords(self, http_response):
        headers, body = http_response.tostring().split("\r\n\r\n", 1)

        # skip non-text responses
        if headers.lower().find("context-type: text") == -1:
            return

        tag_stripper = TagStripper()
        page_text = tag_stripper.strip(body)    
        words = re.findall(r'[a-zA-Z]\w{2,}', page_text)

        for word in words:
            # filter out long strings
            if len(word) <= 12:
                self.wordlist.add(word.lower())
            return
    
    @staticmethod
    def mangle(word):
        year = datetime.now().year
        suffixes = ['', '1', '!', year]
        mangled = []
        for password in (word, word.capitalize()):
            for suffix in suffixes:
                mangled.append(f"{password}, {suffix}")
        return mangled
    

    def display_wordlist(self):
        print(f'# BHP Wordlist for site(s) {", ".join(self.hosts)}')
        for word in sorted(self.wordlist):
            for password in self.mangle(word):
                print(password)
        return
```


## **🛠 Code Breakdown**

### **1️⃣ Import Required Modules**
```python
import re
from datetime import datetime
from html.parser import HTMLParser

from burp import IBurpExtender, IContextMenuFactory
from javax.swing import JMenuItem
from java.util import List, ArrayList
from java.net import URL
```
- **`re`** → Regular expressions for extracting words.
- **`datetime`** → Retrieves the current year (used for word mangling).
- **`HTMLParser`** → Strips HTML tags from responses.
- **`Burp Suite APIs`**:
  - `IBurpExtender`: Registers the extension in Burp Suite.
  - `IContextMenuFactory`: Creates a right-click menu in Burp.

---

### **2️⃣ HTML Tag Stripping Class**
```python
class TagStripper(HTMLParser):
    def __init__(self):
        HTMLParser.__init__(self)
        self.page_text = []

    def handleData(self, data):
        self.page_text.append(data)

    def handleComment(self, data):
        self.handle_data(data)

    def strip(self, html):
        self.feed(html)
        return " ".join(self.page_text)
```
- **Extracts only the readable text** from an HTML page.
- **Ignores comments and HTML tags**.

---

### **3️⃣ Burp Extension Registration**
```python
class BurpExtender(IBurpExtender, IContextMenuFactory):
    def registerExtenderCallbacks(self, callbacks):
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        self.context = None
        self.hosts = set()

        # Start with a common password
        self.wordlist = {"password"}

        callbacks.setExtensionName("BHP Wordlist")
        callbacks.registerContextMenuFactory(self)
```
- Registers the extension **"BHP Wordlist"** in Burp Suite.
- **Stores visited hosts in `self.hosts`** to avoid redundancy.
- **`self.wordlist` starts with a default word: `"password"`**.

---

### **4️⃣ Create Context Menu for Wordlist Generation**
```python
    def createMenuItems(self, context_menu):
        self.context = context_menu
        menu_list = ArrayList()
        menu_list.add(JMenuItem("Create Wordlist", actionPerformed=self.wordlist_menu))
        return menu_list
```
- Adds a **right-click menu option** in Burp Suite called **"Create Wordlist"**.

---

### **5️⃣ Extract Words from HTTP Responses**
```python
    def wordlistMenu(self, event):
        # Grab the details of what the user clicked
        http_traffic = self.context.getSelectedMessages()

        for traffic in http_traffic:
            http_service = traffic.getHttpService()
            host = http_service.getHost()
            self.hosts.add(host)
            http_response = traffic.getResponse()
            if http_response:
                self.getWords(http_response)
        self.display_wordlist()
        return
```
- **Loops through selected HTTP responses**.
- **Extracts the host** and **adds it to the set of visited hosts**.
- **Passes the HTTP response to `getWords()`** for word extraction.

---

### **6️⃣ Extract Words from the Webpage**
```python
    def getWords(self, http_response):
        headers, body = http_response.tostring().split("\r\n\r\n", 1)

        # Skip non-text responses
        if headers.lower().find("content-type: text") == -1:
            return

        tag_stripper = TagStripper()
        page_text = tag_stripper.strip(body)    
        words = re.findall(r'[a-zA-Z]\w{2,}', page_text)

        for word in words:
            # Filter out long strings
            if len(word) <= 12:
                self.wordlist.add(word.lower())
```
- **Extracts words from the HTTP response body**.
- **Ignores responses that are not text-based**.
- **Filters out words longer than 12 characters** to keep the wordlist concise.

---

### **7️⃣ Word Mangling for Password Guessing**
```python
    @staticmethod
    def mangle(word):
        year = datetime.now().year
        suffixes = ['', '1', '!', year]
        mangled = []
        for password in (word, word.capitalize()):
            for suffix in suffixes:
                mangled.append(f"{password}{suffix}")
        return mangled
```
- **Creates password variations** for each extracted word:
  - **Capitalizes** the first letter.
  - **Appends common suffixes** (`1`, `!`, and the current year).

---

### **8️⃣ Display the Generated Wordlist**
```python
    def display_wordlist(self):
        print(f'# BHP Wordlist for site(s) {", ".join(self.hosts)}')
        for word in sorted(self.wordlist):
            for password in self.mangle(word):
                print(password)
        return
```
- **Displays the generated wordlist**.
- **Prints password variations for each extracted word**.

---

## **🚀 How to Run the Extension in Burp Suite**
### **1️⃣ Install Jython in Burp Suite**
- Go to **Extender** → **Options**.
- Scroll down to **Python Environment**.
- Click **Select file** and choose the **Jython standalone JAR** file.

### **2️⃣ Load the Extension**
- Go to **Extender** → **Extensions** → **Add**.
- Select **Extension Type: Python**.
- Click **Select file** and choose this Python script.

### **3️⃣ Using the Wordlist Generator**
1. **Open Burp Suite's Proxy or Repeater tab**.
2. **Right-click on an HTTP request**.
3. **Click "Create Wordlist"**.
4. **Check the Burp Suite console for the generated wordlist**.

---

## **🔥 Why is This Useful?**
✔ **Extracts real-world words from target websites**.  
✔ **Generates a password wordlist automatically**.  
✔ **Creates password variations for effective password attacks**.  
✔ **Integrates directly into Burp Suite** for seamless pentesting.  
