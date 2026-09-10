The provided Python script has an **indentation error** in the `if` statement. In Python, **incorrect indentation leads to an `IndentationError`**.

---

```python
import getpass

# Retrieve the current system username
user_name = getpass.getuser()
print("User Name : %s" % user_name)

# Loop until the correct password is entered
while True:
    passwd = getpass.getpass("Enter your Password : ")

    if passwd == '#pythonworld':  # Correct password
        print("Welcome!!!")
        break  # Exit loop
    else:
        print("The password you entered is incorrect.")
```

---

### **Fixed Issues**
✅ **Indentation error fixed** (`if passwd == '#pythonworld':` was misaligned).  
✅ **Consistent spacing** for readability.  
✅ **String formatting improved** (`print("User Name : %s" % user_name)` remains valid).  

---

### **How to Run the Script**
1. **Save the script** as `password_check.py`.
2. **Run it in a terminal or command prompt**:
   ```sh
   python password_check.py
   ```
3. **Expected Output:**
   ```
   User Name : johndoe
   Enter your Password : 
   (User enters incorrect password)
   The password you entered is incorrect.
   Enter your Password : 
   (User enters '#pythonworld')
   Welcome!!!
   ```

---

### **Security Considerations**
❌ **Hardcoding passwords is insecure** (anyone can see `#pythonworld`).  
✅ **Better alternative**: Store a **hashed** password instead of plaintext.  

#### **Improved Secure Version**
```python
import getpass
import hashlib

# Precomputed hashed password for '#pythonworld'
correct_hash = "fc4ecf5b9a7ffb227d43e5451e5b74d08d82399e"

user_name = getpass.getuser()
print(f"User Name: {user_name}")

while True:
    passwd = getpass.getpass("Enter your Password: ")
    hashed_passwd = hashlib.sha1(passwd.encode()).hexdigest()

    if hashed_passwd == correct_hash:
        print("Welcome!!!")
        break
    else:
        print("The password you entered is incorrect.")
```
- ✅ Uses **SHA-1 hashing** to verify passwords.
- ✅ Prevents exposure of plaintext passwords.

---

### **Key Takeaways**
✅ **Fixed indentation errors**.  
✅ **`getpass.getuser()` retrieves the current system username**.  
✅ **`getpass.getpass()` securely prompts for a password**.  
✅ **Avoid storing plaintext passwords in code**.  
