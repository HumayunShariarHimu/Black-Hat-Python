# **PCAP Image Extractor & Face Detector**

This Python script extracts images from HTTP traffic inside a **PCAP (packet capture) file**, saves them, and then attempts **face detection** using OpenCV.

```python
import cv2
import re
import zlib
from kamene.all import *

pictures_directory = "pic_carver/pictures"
faces_directory = "pic_carver/faces"
pcap_file = "bhp.pcap"

def face_detect(path, file_name):
    img = cv2.imread(path)
    cascade = cv2.CascadeClassifier("haarcascade_frontalface_alt.xml")  # Fixed filename
    rects = cascade.detectMultiScale(img, 1.3, 4, cv2.CASCADE_SCALE_IMAGE, (20,20))

    if len(rects) == 0:
        return False
    rects[:, 2] += rects[:, :2]

    # Highlights faces in the image
    for x1, y1, x2, y2 in rects:
        cv2.rectangle(img, (x1, y1), (x2, y2), (127, 255, 0), 2)
        cv2.imwrite(f"{faces_directory}/{pcap_file}_{file_name}", img)  # Fixed filename format
    return True

def get_http_headers(http_payload):
    try:
        headers_raw = http_payload[:http_payload.index(b"\r\n\r\n") + 2]  # Fixed byte handling
        headers = dict(re.findall(rb"(?P<name>.*?): (?P<value>.*?)\r\n", headers_raw))
        headers = {k.decode(): v.decode() for k, v in headers.items()}  # Decode bytes to strings
    except:
        return None
    if "Content-Type" not in headers:
        return None
    return headers

def extract_image(headers, http_payload):
    image = None
    image_type = None

    try:
        if "image" in headers["Content-Type"]:
            image_type = headers["Content-Type"].split("/")[1]
            image = http_payload[http_payload.index(b"\r\n\r\n") + 4:]  # Fixed byte slicing
            
            # Try to decompress the image if needed
            try:
                if "Content-Encoding" in headers:
                    if headers["Content-Encoding"] == "gzip":
                        image = zlib.decompress(image, 16 + zlib.MAX_WBITS)
                    elif headers["Content-Encoding"] == "deflate":
                        image = zlib.decompress(image)
            except:
                pass
    except:
        return None, None

    return image, image_type

def http_assembler(pcap_fl):
    carved_images = 0
    faces_detected = 0

    a = rdpcap(pcap_fl)
    sessions = a.sessions()

    for session in sessions:
        http_payload = b""  # Fixed to handle bytes
        for packet in sessions[session]:
            try:
                if packet[TCP].dport == 80 or packet[TCP].sport == 80:
                    http_payload += bytes(packet[TCP].payload)  # Fixed encoding
            except:
                pass

        headers = get_http_headers(http_payload)

        if headers is None:
            continue

        image, image_type = extract_image(headers, http_payload)

        if image is not None and image_type is not None:
            file_name = f"{pcap_fl}-pic_carver_{carved_images}.{image_type}"
            with open(f"{pictures_directory}/{file_name}", 'wb') as fd:  # Fixed filename format
                fd.write(image)
            carved_images += 1

            # Now attempt face detection
            try:
                result = face_detect(f"{pictures_directory}/{file_name}", file_name)
                if result:
                    faces_detected += 1
            except:
                pass

    return carved_images, faces_detected

carved_img, faces_dtct = http_assembler(pcap_file)

print(f"Extracted: {carved_img} images")
print(f"Detected: {faces_dtct} faces")

```


## **🔹 Features**
✔ Extracts images from HTTP traffic in a **PCAP file**  
✔ Saves the extracted images to a specified directory  
✔ Uses **OpenCV Haar Cascade** to detect faces in the extracted images  
✔ Saves images with detected faces in a separate folder  

---

## **📌 Code Breakdown**
### **1️⃣ Import Required Modules**
```python
import cv2
import re
import zlib
from kamene.all import *
```
- **cv2** → OpenCV library for image processing  
- **re** → Regular expressions for extracting HTTP headers  
- **zlib** → Decompression of compressed HTTP responses  
- **kamene.all** → Packet capture analysis (alternative to Scapy)  

---

### **2️⃣ Define Directories & PCAP File**
```python
pictures_directory = "pic_carver/pictures"
faces_directory = "pic_carver/faces"
pcap_file = "bhp.pcap"
```
- `pictures_directory` → Folder where extracted images are saved  
- `faces_directory` → Folder where images with detected faces are saved  
- `pcap_file` → The **PCAP file** containing network traffic  

---

### **3️⃣ Face Detection Function**
```python
def face_detect(path, file_name):
    img = cv2.imread(path)
    cascade = cv2.CascadeClassifier("haarcascade_frontalface_alt.xml")  # Face detection model
    rects = cascade.detectMultiScale(img, 1.3, 4, cv2.CASCADE_SCALE_IMAGE, (20,20))

    if len(rects) == 0:
        return False
    rects[:, 2] += rects[:, :2]

    # Highlight detected faces
    for x1, y1, x2, y2 in rects:
        cv2.rectangle(img, (x1, y1), (x2, y2), (127, 255, 0), 2)
        cv2.imwrite(f"{faces_directory}/{pcap_file}_{file_name}", img)  
    return True
```
- Reads an **image file**  
- Uses **Haar Cascade** (`haarcascade_frontalface_alt.xml`) to detect faces  
- Draws a rectangle around detected faces  
- Saves the updated image in the **faces directory**  

---

### **4️⃣ Extract HTTP Headers**
```python
def get_http_headers(http_payload):
    try:
        headers_raw = http_payload[:http_payload.index(b"\r\n\r\n") + 2]  
        headers = dict(re.findall(rb"(?P<name>.*?): (?P<value>.*?)\r\n", headers_raw))
        headers = {k.decode(): v.decode() for k, v in headers.items()}  
    except:
        return None
    if "Content-Type" not in headers:
        return None
    return headers
```
- Extracts **HTTP headers** from the packet payload  
- Converts **binary data to strings**  
- Returns a **dictionary** containing headers  

---

### **5️⃣ Extract Image from HTTP Response**
```python
def extract_image(headers, http_payload):
    image = None
    image_type = None

    try:
        if "image" in headers["Content-Type"]:
            image_type = headers["Content-Type"].split("/")[1]
            image = http_payload[http_payload.index(b"\r\n\r\n") + 4:]  
            
            # Try decompression if needed
            try:
                if "Content-Encoding" in headers:
                    if headers["Content-Encoding"] == "gzip":
                        image = zlib.decompress(image, 16 + zlib.MAX_WBITS)
                    elif headers["Content-Encoding"] == "deflate":
                        image = zlib.decompress(image)
            except:
                pass
    except:
        return None, None

    return image, image_type
```
- Checks if the payload contains an **image**  
- Extracts the **image data**  
- Decompresses the image if it is **gzip** or **deflate** encoded  

---

### **6️⃣ Process PCAP File & Extract Images**
```python
def http_assembler(pcap_fl):
    carved_images = 0
    faces_detected = 0

    a = rdpcap(pcap_fl)
    sessions = a.sessions()

    for session in sessions:
        http_payload = b""  
        for packet in sessions[session]:
            try:
                if packet[TCP].dport == 80 or packet[TCP].sport == 80:
                    http_payload += bytes(packet[TCP].payload)  
            except:
                pass

        headers = get_http_headers(http_payload)

        if headers is None:
            continue

        image, image_type = extract_image(headers, http_payload)

        if image is not None and image_type is not None:
            file_name = f"{pcap_fl}-pic_carver_{carved_images}.{image_type}"
            with open(f"{pictures_directory}/{file_name}", 'wb') as fd:  
                fd.write(image)
            carved_images += 1

            # Try face detection
            try:
                result = face_detect(f"{pictures_directory}/{file_name}", file_name)
                if result:
                    faces_detected += 1
            except:
                pass

    return carved_images, faces_detected
```
- Reads **PCAP file** and extracts **HTTP traffic**  
- Calls `extract_image()` to extract images from HTTP responses  
- Saves the extracted images in `pictures_directory`  
- Calls `face_detect()` to check for faces in the images  

---

### **7️⃣ Run the Script**
```python
carved_img, faces_dtct = http_assembler(pcap_file)

print(f"Extracted: {carved_img} images")
print(f"Detected: {faces_dtct} faces")
```
- Calls `http_assembler()` to process the **PCAP file**  
- Prints the **number of images extracted & faces detected**  

---

## **🚀 How to Run the Script**
### **📌 Prerequisites**
Before running the script, ensure you have the required dependencies installed:

```bash
pip install opencv-python kamene
```

### **📌 Run the Script**
```bash
python script.py
```
(where `script.py` is the name of your file)

### **📌 Required Files**
- **`bhp.pcap`** → The PCAP file containing HTTP traffic  
- **`haarcascade_frontalface_alt.xml`** → Haar Cascade XML for face detection (Download from OpenCV)  

---

## **🎯 Expected Output**
```bash
Extracted: 5 images
Detected: 3 faces
```
- This means **5 images were extracted**, and **3 had faces** detected.  

---

## **🎭 Summary**
✔ Captures **HTTP images** from a **PCAP file**  
✔ Extracts images & **saves them**  
✔ Detects **faces** in extracted images  
✔ Uses **OpenCV** for face detection  
✔ Works with **compressed (gzip/deflate) images**  

---

## **🔹 Next Steps**
💡 Improve face detection accuracy using **deep learning (DNN)** instead of Haar Cascades  
💡 Enhance **error handling** for missing files or corrupt packets  
💡 Add **HTTPS support** with `mitmproxy` to capture encrypted images  

