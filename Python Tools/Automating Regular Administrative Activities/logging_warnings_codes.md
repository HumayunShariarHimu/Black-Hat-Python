This script demonstrates how Python's `logging` module and `warnings` module interact. It shows how to capture warnings and redirect them to logs.

---

### **Breakdown of the Code**
```python
import logging
import warnings
```
- **`logging`** → Manages log messages.
- **`warnings`** → Issues runtime warnings.

---

### **Configuring Logging**
```python
logging.basicConfig(level=logging.INFO,)
```
- **Configures logging** to capture messages of level `INFO` and higher.
- **By default, warnings are not logged**.

---

### **Issuing a Warning (Not Sent to Logs)**
```python
warnings.warn('This warning is not sent to the logs')
```
- **This warning appears on the console but is NOT logged**.

---

### **Capturing Warnings into Logs**
```python
logging.captureWarnings(True)
```
- **Redirects warnings to the logging system**.
- **Any future warnings will be logged**.

---

### **Issuing a Warning (Sent to Logs)**
```python
warnings.warn('This warning is sent to the logs')
```
- This warning **will be logged**.

---

### **Expected Output**
#### **Console Output**
```
/path/to/script.py:8: UserWarning: This warning is not sent to the logs
  warnings.warn('This warning is not sent to the logs')
```
- The first warning is displayed in the console.

#### **Log Output**
```
WARNING:py.warnings:/path/to/script.py:10: UserWarning: This warning is sent to the logs
  warnings.warn('This warning is sent to the logs')
```
- The second warning is captured and logged.

---

### **Improved Version (Logging to a File)**
```python
import logging
import warnings

LOG_FILENAME = 'warnings.log'

# Configure logging to write warnings to a file
logging.basicConfig(filename=LOG_FILENAME, level=logging.INFO, format="%(levelname)s:%(message)s")

warnings.warn('This warning is not sent to the logs')  # Not logged

logging.captureWarnings(True)  # Redirect warnings to logs

warnings.warn('This warning is sent to the logs')  # Logged

print(f"Warnings are logged in {LOG_FILENAME}")
```

---

### **How to Run the Script**
1. **Save the script** as `capture_warnings.py`.
2. **Run it**:
   ```sh
   python capture_warnings.py
   ```
3. **Check the log file**:
   ```sh
   cat warnings.log  # Linux/macOS
   type warnings.log  # Windows
   ```

---

### **Key Takeaways**
✅ **By default, warnings are NOT logged**.  
✅ **`logging.captureWarnings(True)` redirects warnings to logs**.  
✅ **Use `logging.basicConfig(filename="file.log")` to save warnings to a file**.  
