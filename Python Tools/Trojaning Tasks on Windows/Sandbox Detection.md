# Sandbox Detection Script

This script is designed to detect whether it's running inside a sandbox environment by monitoring user input events (keyboard and mouse). If it doesn't detect normal user behavior (like keystrokes or mouse clicks), it may indicate the code is running in a controlled or virtual environment (a sandbox). Here's a breakdown of how the code works:

#### Libraries Used:
- **`ctypes`**: Allows interaction with Windows APIs to check user input and system time.
- **`random`**: Used to generate random values for thresholds.
- **`time`**: Handles time-related operations, like checking how long since the last user input.
- **`sys`**: Provides system-specific parameters and functions, such as exiting the program.

#### Key Functions and Classes:

1. **`LastInputInfo` (ctypes.Structure)**: This structure is used to retrieve the time of the last user input (keyboard or mouse).

2. **`get_last_input()`**: This function calls the Windows API to determine the time of the last input event and calculates the elapsed time since then. If the machine hasn't received input in a while, it might be running in a sandbox.

3. **`get_key_press()`**: Monitors keyboard and mouse input. If a key is pressed or a mouse click is detected, it updates the counters (`keystrokes`, `mouse_clicks`). A left mouse click triggers a timestamp for further analysis.

4. **`detect_sandbox()`**: This function monitors user input in a loop, counting keystrokes, mouse clicks, and double-clicks. It checks whether the behavior is consistent with a normal user or if it's too automated, which may indicate the script is running in a sandbox.

    - **Detection Conditions**:
        - It ensures there is a reasonable amount of user input (keystrokes, mouse clicks, and double-clicks).
        - If there is insufficient input (i.e., all thresholds are exceeded), the script exits.
        - The script can exit early if the time since the last input exceeds a certain threshold (indicating inactivity).
    
5. **Exit Conditions**:
    - If a certain number of keystrokes, mouse clicks, and double-clicks are not reached, the script assumes it is not in a sandbox and terminates with `sys.exit()`.
    - If the system has been inactive for a long time (`max_input_threshold`), it also exits.

#### How to Run:

To run this script:

1. Ensure you have a Python environment set up.
2. Save the script to a file, e.g., `sandbox_detector.py`.
3. Run the script using Python from the command line or an IDE:
   ```bash
   python sandbox_detector.py
   ```
   
If the script detects enough user input, it will print "We are ok!" and finish running. If it doesn't detect the behavior of a typical user, it will exit early.

### Code:

```python
import ctypes
import random
import time
import sys

# Windows API handles
user32 = ctypes.windll.user32
kernel32 = ctypes.windll.kernel32

# Input counters
keystrokes = 0
mouse_clicks = 0
double_clicks = 0

# Structure for getting last input info from the system
class LastInputInfo(ctypes.Structure):
    _fields_ = [("cbSize", ctypes.c_uint), ("dwTime", ctypes.c_ulong)]

# Function to get the time of the last input
def get_last_input():
    struct_lastinputinfo = LastInputInfo()
    struct_lastinputinfo.cbSize = ctypes.sizeof(LastInputInfo)

    # Get last input registered
    user32.GetLastInputInfo(ctypes.byref(struct_lastinputinfo))

    # Determine how long the machine has been running
    run_time = kernel32.GetTickCount()
    elapsed = run_time - struct_lastinputinfo.dwTime
    print(f"[*] It's been {elapsed} milliseconds since the last input event")
    return elapsed

# Function to check for key presses and mouse clicks
def get_key_press():
    global mouse_clicks
    global keystrokes

    # Loop through all possible virtual keys (0-255)
    for code in range(0, 0xff):
        if user32.GetAsyncKeyState(code) == -32767:
            # 0x1 is the code for left mouse click
            if code == 1:
                mouse_clicks += 1
                return time.time()
            else:
                keystrokes += 1
    return None

# Function to detect if the script is running in a sandbox environment
def detect_sandbox():
    global mouse_clicks
    global keystrokes

    # Randomize max keystrokes and mouse clicks thresholds
    max_keystrokes = random.randint(10, 25)
    max_mouse_clicks = random.randint(5, 25)

    # Double-click detection variables
    double_clicks = 0
    max_double_clicks = 10
    double_clicks_threshold = 0.250
    first_double_click = None

    average_mousetime = 0  # Variable never used but defined

    max_input_threshold = 30000  # Threshold for maximum input inactivity

    previous_timestamp = None
    detection_complete = False

    last_input = get_last_input()

    # Exit early if we hit our inactivity threshold
    if last_input >= max_input_threshold:
        sys.exit()

    # Loop until detection is complete
    while not detection_complete:
        keypress_time = get_key_press()
        if keypress_time is not None and previous_timestamp is not None:

            # Calculate the time between double clicks
            elapsed = keypress_time - previous_timestamp

            # If a double-click occurred
            if elapsed <= double_clicks_threshold:
                double_clicks += 1

                if first_double_click is None:
                    first_double_click = time.time()
                else:
                    # If a rapid succession of double-clicks occurred
                    if double_clicks == max_double_clicks:
                        if keypress_time - first_double_click <= (max_double_clicks * double_clicks_threshold):
                            sys.exit()

            # If enough user input is detected, we stop
            if keystrokes >= max_keystrokes and double_clicks >= max_double_clicks and mouse_clicks >= max_mouse_clicks:
                return
            previous_timestamp = keypress_time

        elif keypress_time is not None:
            previous_timestamp = keypress_time

# Call the sandbox detection function
detect_sandbox()
print("We are ok!")
```

### Key Points:
- The script checks for user activity (mouse clicks, keystrokes) to decide if it's being run in a sandbox.
- If no normal user behavior is detected within the specified thresholds, it exits.
- It uses a combination of system time, user input monitoring, and randomness to determine whether it's in a typical environment.

          
