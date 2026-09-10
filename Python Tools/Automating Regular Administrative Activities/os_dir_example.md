This script prompts the user to enter their system **username** and a **password**, granting access only if the correct password is entered.

---

### **Breakdown of the Code**
```python
import getpass  # Import the getpass module for secure user input
```
- **`getpass`** is used to **retrieve the current system username** and **securely input a password** (hiding input from the screen).

---

### **Fetching the System Username**
```python
user_name = getpass.getuser()
print("User Name : %s" % user_name)
```
- **`getpass.getuser()`** retrieves the currently logged-in **system username**.
- The username is displayed using **formatted string output**.

---

### **Password Authentication Loop**
```python
while True:
    passwd = getpass.getpass("Enter your Password : ")
```
- **Loops indefinitely** until the correct password is entered.
- **`getpass.getpass()`** securely prompts for a password **(input remains hidden).**

---

### **Checking the Password**
```python
    if passwd == '#pythonworld':  # Correct password
        print("Welcome!!!")
        break  # Exit the loop
    else:
        print("The password you entered is incorrect.")
```
- **If the correct password (`#pythonworld`) is entered**, the script prints `"Welcome!!!"` and **exits**.
- **Otherwise**, it prints an error message and **re-prompts** the user.

---

### **Corrected Version (Fixing Indentation)**
The original code had an **indentation error** in the `if` statement. Here’s the fixed version:

```python
import getpass

# Retrieve the system's username
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

### **How to Run the Script**
1. **Save the script** as `auth.py`.
2. **Run it in a terminal**:
   ```sh
   python auth.py
   ```
3. **Expected Output:**
   ```
   User Name : johndoe
   Enter your Password :
   (Hidden input)
   The password you entered is incorrect.
   Enter your Password :
   (Hidden input)
   Welcome!!!
   ```

---

### **Security Considerations**
❌ **Hardcoding passwords (`#pythonworld`) is insecure**.  
✅ **Use hashed passwords** or environment variables for security.  
✅ **Never print user passwords** in production systems.  

---

### **Enhanced Version (Using Hashed Passwords)**
For improved security, use **hashed passwords** instead of plaintext:

```python
import getpass
import hashlib

# Stored hashed password (hashed version of '#pythonworld')
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
- ✅ Uses **SHA-1 hashing** instead of storing plaintext passwords.  
- ✅ Prevents password exposure in memory.  

---

### **Key Takeaways**
✅ **`getpass.getuser()` retrieves the current system username**.  
✅ **`getpass.getpass()` securely prompts for a password**.  
✅ **Fixed indentation issues in the original code**.  
✅ **Hash passwords instead of storing them in plaintext**.  
