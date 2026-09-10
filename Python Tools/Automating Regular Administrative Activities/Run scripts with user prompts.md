This script **securely prompts the user for a password** using Python’s `getpass` module. It ensures that the password input is **not echoed** on the screen for security reasons.

```python
import getpass
try:
	p = getpass.getpass("Enter your password: ")
except Exception as error:
	print('ERROR', error)
else:
	print('Password entered:', p)
```

---

## **Breakdown of the Code**
### **1. Import Required Module**
```python
import getpass
```
- `getpass` provides a **secure way to prompt the user for input** without displaying it.

---

### **2. Try to Get User Input Securely**
```python
try:
	p = getpass.getpass("Enter your password: ")
```
- `getpass.getpass(prompt)`:
  - Displays the prompt `"Enter your password: "`.
  - **Hides the input characters** (useful for password entry).
  - Works in both **CLI and terminal** but **may fail in some IDEs** (e.g., Jupyter Notebook).

---

### **3. Handle Exceptions**
```python
except Exception as error:
	print('ERROR', error)
```
- If `getpass.getpass()` fails (e.g., **running in an unsupported environment**), it **catches the exception** and prints an error message.

---

### **4. Print the Entered Password (For Testing)**
```python
else:
	print('Password entered:', p)
```
- **For security reasons, you should not print passwords in real applications**.
- This line is just for testing purposes.

---

## **How to Run the Script**
### ✅ **Run in Terminal or Command Line**
```sh
python script.py
```
#### **Expected Output**
```
Enter your password: (input hidden)
Password entered: *********  # Input is hidden but stored
```

---

## **Potential Issues**
### ⚠️ **Does Not Work in Some IDEs**
- `getpass.getpass()` **may fail in Jupyter Notebook, VS Code terminal, or IDLE**.
- Instead, use **plain `input()`** for debugging:
  ```python
  p = input("Enter your password: ")
  ```
  - ❌ **Not secure** (password is visible).
  - ✅ **Works in all environments**.

---

## **Key Takeaways**
✅ **Uses `getpass` for secure password entry**.  
✅ **Prevents passwords from being displayed on screen**.  
✅ **Handles exceptions if `getpass` fails**.  
✅ **Should not print the password in real applications**.  

---
