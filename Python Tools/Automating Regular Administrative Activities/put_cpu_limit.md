This script **sets a CPU time limit** using the `resource` module and **terminates the process** if it exceeds the allocated CPU time.

---

```python
import resource
import sys
import signal
import time
def time_expired(n, stack):
	print('EXPIRED :', time.ctime())
	raise SystemExit('(time ran out)')
signal.signal(signal.SIGXCPU, time_expired)
# Adjust the CPU time limit
soft, hard = resource.getrlimit(resource.RLIMIT_CPU)
print('Soft limit starts as  :', soft)
resource.setrlimit(resource.RLIMIT_CPU, (10, hard))
soft, hard = resource.getrlimit(resource.RLIMIT_CPU)
print('Soft limit changed to :', soft)
print()
# Consume some CPU time in a pointless exercise
print('Starting:', time.ctime())
for i in range(200000):
	for i in range(200000):
		v = i * i
# We should never make it this far
print('Exiting :', time.ctime())
```

## **Breakdown of the Code**
### **1. Import Required Modules**
```python
import resource
import sys
import signal
import time
```
- `resource` → Controls system resource limits.  
- `sys` → Provides system-specific functions.  
- `signal` → Handles OS-level signals (e.g., CPU time exceeded).  
- `time` → Used for tracking timestamps.

---

### **2. Define a Signal Handler for CPU Time Expiry**
```python
def time_expired(n, stack):
    print('EXPIRED :', time.ctime())  # Print the expiration time
    raise SystemExit('(time ran out)')  # Exit the program when time is exceeded
```
- This function is **triggered when CPU time exceeds the limit**.
- It prints an expiration message and **forcefully exits the program**.

---

### **3. Register Signal Handler**
```python
signal.signal(signal.SIGXCPU, time_expired)
```
- `SIGXCPU` is a **signal sent when CPU time is exceeded**.
- This line registers `time_expired` to handle `SIGXCPU`.

---

### **4. Adjust CPU Time Limit**
```python
soft, hard = resource.getrlimit(resource.RLIMIT_CPU)
print('Soft limit starts as  :', soft)
resource.setrlimit(resource.RLIMIT_CPU, (10, hard))  # Set CPU time limit to 10 seconds
soft, hard = resource.getrlimit(resource.RLIMIT_CPU)
print('Soft limit changed to :', soft)
```
- Retrieves the **current soft and hard CPU limits**.
- Sets a **soft CPU time limit** of **10 seconds**.
- Prints the **updated CPU time limit**.

---

### **5. Simulate CPU-Intensive Task**
```python
print('Starting:', time.ctime())
for i in range(200000):
    for j in range(200000):
        v = i * i
```
- Runs **nested loops** to **consume CPU time**.
- This operation will likely **trigger the time limit** and **terminate** before completion.

---

### **6. Exit Message (May Never Execute)**
```python
print('Exiting :', time.ctime())
```
- If the script **somehow survives beyond the time limit**, it prints an exit message.
- However, **this will likely never execute** due to the time restriction.

---

## **Expected Output**
### ✅ **Before Expiry**
```
Soft limit starts as  : -1
Soft limit changed to : 10

Starting: Mon Feb  5 12:34:56 2024
```
(The program runs until CPU time exceeds 10 seconds.)

---

### ❌ **On Time Expiry**
```
EXPIRED : Mon Feb  5 12:35:06 2024
(time ran out)
```
- When the CPU time **exceeds 10 seconds**, the `time_expired` function is triggered.
- The script **terminates immediately**.

---

## **How to Run the Script**
1. **Save the script** as `cpu_limit.py`
2. **Run it in a Unix-based system (Linux/macOS)**:
   ```sh
   python cpu_limit.py
   ```
3. **Observe the CPU time expiration** after 10 seconds.

---

## **Key Takeaways**
✅ **`resource.setrlimit(resource.RLIMIT_CPU, (10, hard))` limits CPU time to 10 seconds**.  
✅ **`signal.signal(signal.SIGXCPU, handler)` catches CPU time limit breaches**.  
✅ **CPU-intensive loops simulate high processing load**.  
✅ **The script terminates when the time limit is exceeded**.  
