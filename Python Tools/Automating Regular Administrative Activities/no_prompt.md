This script securely prompts the user to enter a password using `getpass.getpass()`, handling potential exceptions.

---

### **Breakdown of the Code**
```python
import getpass  # Import the getpass module for secure password input
```
- `getpass` allows **hidden password entry** (characters are not displayed while typing).

---

### **Password Input with Exception Handling**
```python
try:
    p = getpass.getpass()
```
- **Prompts the user** to enter a password.
- **Hides the input** (prevents it from being displayed on the screen).
- **Raises an exception** if running in an unsupported environment (e.g., non-interactive shell).

---

### **Handling Exceptions**
```python
except Exception as error:
    print('ERROR', error)
```
- **Catches errors**, such as:
  - Running in a script without terminal input.
  - Permission issues.

---

### **If No Errors, Print Password**
```python
else:
    print('Password entered:', p)
```
- **If no exception occurs**, the entered password is printed.

---

### **Expected Output**
#### ✅ **Normal Execution**
```
Password: (User enters input but it remains hidden)
Password entered: mysecret
```
(The actual password is not shown while typing.)

#### ❌ **Error Scenario (e.g., Running in an Unattended Script)**
```
ERROR getpass was called, but this environment does not support input requests.
```

---

### **Security Considerations**
❌ **DO NOT print the password in production**  
✅ **Use password input for authentication, not display**  

**Safer Version (Masking Output)**
```python
import getpass

try:
    p = getpass.getpass("Enter your password: ")
except Exception as error:
    print("ERROR:", error)
else:
    print("Password received successfully!")  # No exposure of the password
```

---

### **How to Run the Script**
1. **Save the script** as `password_input.py`.
2. **Run it in the terminal**:
   ```sh
   python password_input.py
   ```
3. **Enter a password when prompted** (it will not be visible as you type).

---

### **Key Takeaways**
✅ `getpass.getpass()` securely collects passwords.  
✅ **Exceptions must be handled** in case of unsupported environments.  
✅ **Never print passwords in real-world applications**.  
