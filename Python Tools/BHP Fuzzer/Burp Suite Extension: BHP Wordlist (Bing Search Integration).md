# **Burp Suite Extension: BHP Wordlist (Bing Search Integration)**  
This Burp Suite extension integrates **Bing Search** to find **subdomains, related websites, and IP-based domains** for a given target. It enhances penetration testing by automating reconnaissance.

To create the BING API Key, you may lead to https://www.microsoft.com/en-us/bing/apis/bing-web-search-api

```python
import base64
import json
import re
import socket
import urllib.error
import urllib.parse
import urllib.request

from burp import IBurpExtender
from burp import IContextMenuFactory
from java.net import URL
from java.util import ArrayList
from javax.swing import JMenuItem

bing_api_key = "YOUR_API_KEY"

class BurpExtender(IBurpExtender, IContextMenuFactory):
    def registerExtenderCallbacks(self, callbacks):
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        self.context = None

        #setting up extensions
        callbacks.setExtensionName("BHP Wordlist")
        callbacks.registerContextMenuFactory(self)
        return
    
    def createMenuItems(self, context_menu):
        self.context = context_menu
        menu_list = ArrayList()
        menu_list.add(JMenuItem("Send to Bing", actionPerformed=self.bing_menu))
        return menu_list

    def bingMenu(self, event):
        # grab the details of what the user clicked
        http_traffic = self.context.getSelectedMessages()
        print(f"{len(http_traffic)} requests highlighted")

        for traffic in http_traffic:
            http_service = traffic.getHttpService()
            host = http_service.getHost()
            print(f"User selected host: {host}")
            self.bing_search(host)
        return

    def bingSearch(self, host):
        # check if we have an IP or hostname
        is_ip = re.match(r'[0-9]+(?:\.[0-9]+){3}', host)

        if is_ip:
            ip_address = host
            domain = False
        else:
            ip_address = socket.gethostbyname(host)
            domain = True

        bing_query_string = f"'ip:{ip_address}'" 
        self.bing_query(bing_query_string)

        if domain:
            bing_query_string = f"'domain:{host}'"
            self.bing_query(bing_query_string)    

    def bingQuery(self, bing_query_string):

        print(f"Performing Bing search: {bing_query_string}")

        quoted_query = urllib.parse.quote(bing_query_string)

        http_request = f"GET https://api.datamarket.azure.com/Bing/Search/Web?$format=json&$top=20&Query={quoted_query} HTTP/1.1\r\n"
        http_request += "Host: api.datamarket.azure.com\r\n"
        http_request += "Connection: close\r\n"
        http_request += "Authorization: Basic %s\r\n" % base64.b64encode(":%s" % bing_api_key)
        http_request += "User-Agent: Pentesting Python\r\n\r\n"

        json_body = self._callbacks.makeHttpRequest("api.datamarket.azure.com", 443, True, http_request).tostring()

        json_body = json_body.split("\r\n\r\n", 1)[1]

        try:
            r = json.loads(json_body)
            if len(r["d"]["results"]):
                for site in r["d"]["results"]:
                    print("*" * 100)
                    print(site['Title'])
                    print(site['Url'])
                    print(site['Description'])
                    print("*" * 100)
                    j_url = URL(site['Url'])
                    if not self._callbacks.isInScope(j_url):
                        print("Adding to Burp scope")
                        self._callbacks.includeInScope(j_url)
        except:
            print("No results from Bing")
               
        return
```
---

## **🛠 Code Breakdown**

### **1️⃣ Import Required Modules**
```python
import base64
import json
import re
import socket
import urllib.error
import urllib.parse
import urllib.request

from burp import IBurpExtender, IContextMenuFactory
from java.net import URL
from java.util import ArrayList
from javax.swing import JMenuItem
```
- **`base64, json, re, socket`** → For encoding API keys, parsing responses, regex checks, and resolving IPs.
- **`urllib`** → To handle HTTP requests.
- **`Burp Suite APIs`**:
  - `IBurpExtender`: Registers the extension in Burp Suite.
  - `IContextMenuFactory`: Creates a right-click menu in Burp.

---

### **2️⃣ Define the Bing API Key**
```python
bing_api_key = "YOUR_API_KEY"
```
- Replace `"YOUR_API_KEY"` with a **valid Bing Search API key**.

---

### **3️⃣ Burp Extension Registration**
```python
class BurpExtender(IBurpExtender, IContextMenuFactory):
    def registerExtenderCallbacks(self, callbacks):
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        self.context = None

        # Setting up extensions
        callbacks.setExtensionName("BHP Wordlist")
        callbacks.registerContextMenuFactory(self)
```
- Registers the extension **"BHP Wordlist"** in Burp Suite.
- Implements `registerContextMenuFactory()` to create a **custom right-click menu**.

---

### **4️⃣ Create Context Menu for Bing Search**
```python
    def createMenuItems(self, context_menu):
        self.context = context_menu
        menu_list = ArrayList()
        menu_list.add(JMenuItem("Send to Bing", actionPerformed=self.bing_menu))
        return menu_list
```
- Adds a **right-click option** in Burp Suite called **"Send to Bing"**.

---

### **5️⃣ Handling the Menu Click (Extracting Host)**
```python
    def bingMenu(self, event):
        # Grab the details of what the user clicked
        http_traffic = self.context.getSelectedMessages()
        print(f"{len(http_traffic)} requests highlighted")

        for traffic in http_traffic:
            http_service = traffic.getHttpService()
            host = http_service.getHost()
            print(f"User selected host: {host}")
            self.bing_search(host)
```
- **Retrieves the selected request's host** and **sends it for Bing search**.

---

### **6️⃣ Checking if the Host is an IP or Domain**
```python
    def bingSearch(self, host):
        # Check if we have an IP or hostname
        is_ip = re.match(r'[0-9]+(?:\.[0-9]+){3}', host)

        if is_ip:
            ip_address = host
            domain = False
        else:
            ip_address = socket.gethostbyname(host)
            domain = True

        bing_query_string = f"'ip:{ip_address}'" 
        self.bing_query(bing_query_string)

        if domain:
            bing_query_string = f"'domain:{host}'"
            self.bing_query(bing_query_string)
```
- **Checks if the selected host is an IP or domain**.
- **Resolves domain names to IP addresses** if needed.
- **Generates Bing search queries** for:
  - `ip:<IP>` → Searches for websites hosted on the same IP.
  - `domain:<domain>` → Searches for related domains.

---

### **7️⃣ Querying Bing API**
```python
    def bingQuery(self, bing_query_string):
        print(f"Performing Bing search: {bing_query_string}")

        quoted_query = urllib.parse.quote(bing_query_string)

        http_request = f"GET https://api.datamarket.azure.com/Bing/Search/Web?$format=json&$top=20&Query={quoted_query} HTTP/1.1\r\n"
        http_request += "Host: api.datamarket.azure.com\r\n"
        http_request += "Connection: close\r\n"
        http_request += "Authorization: Basic %s\r\n" % base64.b64encode(":%s" % bing_api_key)
        http_request += "User-Agent: Pentesting Python\r\n\r\n"

        json_body = self._callbacks.makeHttpRequest("api.datamarket.azure.com", 443, True, http_request).tostring()
```
- **Sends a request to Bing API** using Burp Suite’s `makeHttpRequest()`.
- Uses **Basic Authentication** (Base64 encoded API key).

---

### **8️⃣ Parsing Bing Results**
```python
        json_body = json_body.split("\r\n\r\n", 1)[1]

        try:
            r = json.loads(json_body)
            if len(r["d"]["results"]):
                for site in r["d"]["results"]:
                    print("*" * 100)
                    print(site['Title'])
                    print(site['Url'])
                    print(site['Description'])
                    print("*" * 100)

                    j_url = URL(site['Url'])
                    if not self._callbacks.isInScope(j_url):
                        print("Adding to Burp scope")
                        self._callbacks.includeInScope(j_url)
        except:
            print("No results from Bing")
```
- **Parses the Bing search results (JSON)**.
- **Prints the results in the console** (Title, URL, Description).
- **Automatically adds discovered domains to Burp Suite's scope**.

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

### **3️⃣ Using the Bing Search Feature**
1. **Open Burp Suite's Proxy or Repeater tab**.
2. **Right-click on an HTTP request**.
3. **Click "Send to Bing"**.
4. **Check the Burp Suite console for results**.

---

## **🔥 Why is This Useful?**
✔ **Automates reconnaissance** by finding related domains and IPs.  
✔ **Integrates directly into Burp Suite**.  
✔ **Enhances security testing** with Bing Dorking.  
✔ **Automatically adds discovered URLs to Burp's scope** for further testing.  
