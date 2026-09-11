# Black-Hat-Python

> A curated and organized collection of resources related to **Black-Hat-Python**.

**Maintained by [Humayun Shariar Himu](https://github.com/HumayunShariarHimu)**

# Black-Hat-Python
 
![YouTube Channel Subscribers](https://img.shields.io/youtube/channel/subscribers/UClhKVCHjOxBTNM50lOBTgoA)
![Discord](https://img.shields.io/discord/1163365511309049948)

# Contact With Me:

    <img src="https://img.shields.io/static/v1?message=Youtube&logo=youtube&label=&color=FF0000&logoColor=white&labelColor=&style=for-the-badge" height="27" alt="youtube logo"  />
  </a>
    <img src="https://img.shields.io/static/v1?message=Instagram&logo=instagram&label=&color=E4405F&logoColor=white&labelColor=&style=for-the-badge" height="27" alt="instagram logo"  />
  </a>
    <img src="https://img.shields.io/static/v1?message=Twitch&logo=twitch&label=&color=9146FF&logoColor=white&labelColor=&style=for-the-badge" height="27" alt="twitch logo"  />
  </a>
    <img src="https://img.shields.io/static/v1?message=Proton%20Mail&logo=protonmail&label=&color=7341FF&logoColor=white&labelColor=&style=for-the-badge" height="27" alt="proton mail logo"  />
  </a>
    <img src="https://img.shields.io/static/v1?message=LinkedIn&logo=linkedin&label=&color=0077B5&logoColor=white&labelColor=&style=for-the-badge" height="27" alt="linkedin logo"  />
  </a>
    <img src="https://img.shields.io/static/v1?message=Twitter&logo=twitter&label=&color=1DA1F2&logoColor=white&labelColor=&style=for-the-badge" height="27" alt="twitter logo"  />
  </a>

# You can help me by Donating

> [!NOTE]
> Some scripts and portions of the code in this repository are sourced from other open-source projects on GitHub. All original works are credited to their respective authors. If you are the owner of any content here and would like it removed or updated, please reach out directly.

> [!WARNING]
> **Educational & Ethical Use Only**
> These tools are provided strictly for authorized security research and learning. Unauthorized use, distribution, or modification without proper consent is prohibited. By using this repository, you agree to comply with all local and international laws. The author assumes zero liability for any misuse or damage caused by these tools.

---

# Table of Content

## 1. Password Attacks & Credential Attacks

## 2. Python Networking

### Basic Network Clients & Servers

### SSH Tools

### Packet Sniffers & Network Analysis

### Scapy-Based Attacks

### Port Scanning & Host Discovery

### Advanced Scapy Usage

## 3. Network Attacks
- DNS Spoofing / DNS Poisoning

## 4. Web Application Hacking

### Web Path Discovery & Crawling

### Brute Force Attacks

### Web Vulnerability Testing

### Web Scraping & Reconnaissance

## 5. Burp Suite Extensions & Fuzzing

### Burp Fuzzing Tools

## 6. Reconnaissance & OSINT

## 7. Trojaning & Malware Development

### GitHub Command and Control

### Trojaning Tasks on Windows

## 8. Browser Attacks

## 9. Post-Exploitation & Privilege Escalation

### Shells & Backdoors
### Windows Post-Exploitation

### Linux Post-Exploitation

### Persistence & Exfiltration

## 10. Active Directory Attacks

## 11. Evasion & Anti-Forensics
- Payload Obfuscation
- Process Hollowing
- DLL Injection (Windows)
- Log Clearing / Tampering
- File Timestomping
- Polymorphic Shellcode
- Base64 / XOR Payload Encoding

## 12. C2 (Command & Control)
- DNS-Based C2 Channel
- HTTP / HTTPS C2 Beacon
- Slack / Discord C2
- Encrypted C2 with AES
- Heartbeat / Persistence Mechanism

## 13. Vulnerability Scanning & Exploitation

### Scanning & Fingerprinting
- CVE Scanner / Vulnerability Checker
- Service Version Fingerprinting
- Nmap Python Wrapper (python-nmap)

### Exploit Development
- Exploit Development — Buffer Overflow
- Format String Vulnerability Tester
- Custom Fuzzing Framework (TCP / UDP / File / API)
- ROP Chain Generation
- Buffer Overflow Pattern Generator (like pattern_create)
- Shellcode Encoder / Decoder
- Bad Character Finder

## 14. Cryptography Attacks
- Padding Oracle Attack
- ECB Mode Detection
- Frequency Analysis (Classic Ciphers)
- Weak TLS / SSL Detection
- S/MIME & PGP Key Extraction
- Password Hash Identifier
- AES / XOR Custom Encryption & Decryption

## 15. Phishing Tools
- Phishing Email Generator
- Clone Website for Credential Harvesting
- URL Obfuscation Techniques

## 16. Forensics & Memory Analysis

## 17. Container & Cloud Security
- Docker Escape Techniques
- AWS / Azure / GCP Misconfiguration Scanners
- S3 Bucket Enumeration

## 18. Automating Administrative Tasks

### Input & Output Handling

### System Commands & Execution

### File & Directory Operations

### Configuration Management

### Security & Authentication

### Logging & Monitoring

### Resource Management

### Web Operations

---

# Resources

## Books

-   [Violent Python](https://www.elsevier.com/books/violent-python/unknown/978-1-59749-957-6)
-   [Grey Hat Python](http://www.nostarch.com/ghpython.htm)
-   [Black Hat Python](http://www.nostarch.com/blackhatpython)
-   [Python Penetration Testing Essentials](https://github.com/PacktPublishing/Python-Penetration-Testing-Essentials-Second-Edition)
-   [Python for Secret Agents](https://www.packtpub.com/en-us/product/python-for-secret-agents-volume-ii-9781785283406)
-   [Python Web Penetration Testing Cookbook](https://www.packtpub.com/en-us/product/python-web-penetration-testing-cookbook-9781784399900) 
-   [Learning Penetration Testing with Python](https://www.packtpub.com/en-us/product/learning-penetration-testing-with-python-9781785289552)
-   [Python Forensics](http://www.sciencedirect.com/science/book/9780124186767)
-   [The Beginner's Guide to IDAPython](https://leanpub.com/IDAPython-Book)
-   [Python for Offensive PenTest: A Practical Guide to Ethical Hacking and Penetration Testing Using Python](https://www.amazon.com/Python-Offensive-PenTest-practical-penetration/dp/1788838971)

---

# Python Tools for Penetration Testing

## Network Tools

### Core Network Libraries
-   [Scapy](https://scapy.net): Send, sniff, dissect and forge network packets
-   [Impacket](http://oss.coresecurity.com/projects/impacket.html): Craft and decode network packets with support for NMB and SMB
-   [dpkt](https://github.com/kbandla/dpkt): Fast, simple packet creation/parsing with TCP/IP protocol definitions
-   [pypcap](https://github.com/dugsong/pypcap), [Pcapy](https://github.com/helpsystems/pcapy), [Pcapy-NG](https://github.com/stamparm/pcapy-ng), [libpcap](https://pypi.org/project/libpcap/): Python bindings for libpcap
-   [libdnet](https://github.com/ofalk/libdnet/): Low-level networking routines

### Network Reconnaissance & Enumeration
-   [SMBMap](https://github.com/ShawnDEvans/smbmap): Enumerate Samba share drives across an entire domain
-   [AutoRecon](https://github.com/Tib3rius/AutoRecon): Multi-threaded network reconnaissance tool
-   [Habu](https://github.com/portantier/habu): Python network hacking toolkit
-   [Knock Subdomain Scan](https://github.com/guelfoweb/knock): Enumerate subdomains through wordlist
-   [SubBrute](https://github.com/TheRook/subbrute): Fast subdomain enumeration tool
-   [Spoodle](https://github.com/vjex/spoodle): Mass subdomain + poodle vulnerability scanner

### Network Attacks & MITM
-   [Mitm6](https://github.com/fox-it/mitm6): IPv6-based MITM tool
-   [Mallory](https://github.com/intrepidusgroup/mallory): Extensible TCP/UDP man-in-the-middle proxy
-   [Pytbull-NG](https://github.com/netrunn3r/pytbull-ng/): Flexible IDS/IPS testing framework

## Debugging & Reverse Engineering

### Disassemblers & Decompilers
-   [Capstone](http://www.capstone-engine.org/): Lightweight multi-platform disassembly framework
-   [Keystone](http://www.keystone-engine.org): Lightweight multi-platform assembler framework
-   [diStorm](https://github.com/gdabah/distorm): Disassembler library for AMD64
-   [pydasm](https://github.com/jtpereyda/libdasm/tree/master/pydasm): Python interface to libdasm x86 disassembling library

### Dynamic Analysis & Instrumentation
-   [Frida](http://www.frida.re/): Dynamic instrumentation framework
-   [Unicorn Engine](https://www.unicorn-engine.org/): CPU emulator framework
-   [PyEMU](https://github.com/codypierce/pyemu/): Fully scriptable IA-32 emulator

### Platform-Specific Tools
-   [Androguard](https://github.com/androguard/androguard): Reverse engineering and analysis of Android applications
-   [IDAPython](https://github.com/idapython/src): IDA Pro Python integration plugin
-   [Ghidrathon](https://github.com/mandiant/Ghidrathon): Python 3 scripting extension for Ghidra
-   [CHIPSEC](https://github.com/chipsec/chipsec): Platform security analysis framework

### Debugging Frameworks
-   [Paimei](https://github.com/OpenRCE/paimei): Reverse engineering framework with PyDBG, PIDA, pGRAPH
-   [python-ptrace](http://python-ptrace.readthedocs.org/): Debugger using ptrace
-   [PyDbgEng](http://pydbgeng.sourceforge.net/): Python wrapper for Microsoft Windows Debugging Engine

### Binary Analysis
-   [pefile](https://github.com/erocarrera/pefile): Read and work with Portable Executable files
-   [PyBFD](https://github.com/Groundworkstech/pybfd/): Python interface to GNU Binary File Descriptor library

## Fuzzing Tools

-   [afl-python](http://jwilk.net/software/python-afl): American fuzzy lop support for pure-Python code
-   [Sulley](https://github.com/OpenRCE/sulley): Fuzzer development and fuzz testing framework
-   [Peach Fuzzing Platform](https://github.com/MozillaSecurity/peach/): Extensible fuzzing framework
-   [untidy](https://github.com/kbandla/python-untidy/): General purpose XML fuzzer
-   [Powerfuzzer](http://www.powerfuzzer.com/): Highly automated web fuzzer
-   [Construct](http://construct.readthedocs.org/): Library for parsing and building data structures
-   [Fusil](http://fusil.readthedocs.org/): Python library for writing fuzzing programs

## Web Application Testing

### HTTP Clients & Testing
-   [XSStrike](https://github.com/s0md3v/XSStrike): Advanced XSS detection suite
-   [Requests](https://requests.readthedocs.io/): Elegant and simple HTTP library
-   [lxml](http://lxml.de/index.html): Easy-to-use library for processing XML and HTML
-   [HTTPie](http://httpie.org): Human-friendly cURL-like command line HTTP client
-   [Twill](https://twill-tools.github.io/twill/): Command-line web browsing with automated testing support

### Web Testing & Automation
-   [FunkLoad](https://github.com/nuxeo/FunkLoad): Functional and load web tester
-   [spynner](https://github.com/makinacorpus/spynner): Programmatic web browsing with Javascript/AJAX support
-   [mitmproxy](http://mitmproxy.org/): SSL-capable intercepting HTTP proxy
-   [spidy](https://github.com/rivermont/spidy/): Simple command-line web crawler
-   [Waymap](https://github.com/TrixSec/waymap): Web vulnerability scanner for penetration testers

## Forensics & Memory Analysis

-   [Volatility](http://www.volatilityfoundation.org/): Extract digital artifacts from volatile memory (RAM)
-   [Rekall](https://github.com/google/rekall): Memory analysis framework by Google
-   [TrIDLib](http://mark0.net/code-tridlib-e.html): Identify file types from binary signatures

## Malware Analysis

-   [pyew](https://github.com/joxeankoret/pyew): Hexadecimal editor and disassembler for malware analysis
-   [Exefilter](https://github.com/decalage2/exefilter): Filter file formats and detect/remove active content
-   [jsunpack-n](https://github.com/urule99/jsunpack-n): Generic JavaScript unpacker
-   [yara-python](https://github.com/VirusTotal/yara-python): Identify and classify malware samples
-   [phoneyc](https://github.com/honeynet/phoneyc): Pure Python honeyclient implementation
-   [CapTipper](https://github.com/omriher/CapTipper): Analyse HTTP malicious traffic from PCAP files
-   [Cuckoo](https://github.com/cuckoosandbox/cuckoo): Automated malware analysis system
-   [CAPE](https://github.com/kevoreilly/CAPEv2): Malware configuration and payload extraction

## PDF Analysis

-   [pdfminer.six](https://github.com/pdfminer/pdfminer.six): Extract text from PDF files
-   [peepdf-3](https://github.com/digitalsleuth/peepdf-3): Analyse and explore PDF files for malicious content
-   [Didier Stevens' PDF tools](http://blog.didierstevens.com/programs/pdf-tools): Analyse, identify and create PDF files
-   [pyPDF](https://pypdf.readthedocs.io/): Pure Python PDF toolkit

## Security Analysis & Assessment

### Binary & Vulnerability Analysis
-   [Angr](https://github.com/angr/angr): Binary analysis framework for vulnerability research and exploit development
-   [ScoutSuite](https://github.com/nccgroup/ScoutSuite): Multi-cloud security auditing tool

### Active Directory & Windows
-   [Certipy](https://github.com/ly4k/Certipy): Active Directory Certificate Services enumeration and abuse
-   [BloodHound.py](https://github.com/fox-it/BloodHound.py): Python-based BloodHound ingestor for AD security assessment
-   [wmiexec.py](https://github.com/CoreSecurity/impacket/blob/master/examples/wmiexec.py): Execute Powershell commands via WMI

### General Security Tools
-   [Pentestly](https://github.com/praetorian-inc/pentestly): Python and Powershell penetration testing framework
-   [hacklib](https://github.com/leonli96/python-hacklib): Toolkit for hacking enthusiasts

### OSINT & Intelligence
-   [Exomind](https://github.com/jio-gl/exomind): Framework for building decorated graphs and OSINT modules

## Utility Libraries

### Interactive & Visualization
-   [Project Jupyter](https://jupyter.org): Enhanced interactive shell
-   [matplotlib](https://matplotlib.org): 2D plotting of arrays
-   [Mayavi](http://code.enthought.com/projects/mayavi/): 3D scientific data visualization
-   [RTGraph3D](http://www.secdev.org/projects/rtgraph3d/): Dynamic 3D graphs

### Data Processing & Analysis
-   [Beautiful Soup](http://www.crummy.com/software/BeautifulSoup/): HTML parser optimized for screen-scraping
-   [Pandas](http://pandas.pydata.org/): High-performance data structures and analysis tools
-   [NetworkX](https://networkx.org): Graph library for edges and nodes
-   [Whoosh](https://github.com/whoosh-community/whoosh): Full-text indexing and searching library

### Networking & Communication
-   [Twisted](http://twistedmatrix.com/): Event-driven networking engine
-   [Suds](https://github.com/suds-community/suds): Lightweight SOAP client

### Parsing & Processing
-   [simplejson](https://github.com/simplejson/simplejson/): JSON encoder/decoder
-   [pyparsing](https://pypi.org/project/pyparsing/): General parsing module
-   [lxml](http://lxml.de/): Feature-rich library for XML and HTML
-   [Hachoir](https://hachoir.readthedocs.io/en/latest/): View and edit binary stream field by field

### Automation & Control
-   [Pexpect](https://github.com/pexpect/pexpect): Control and automate other programs
-   [SikuliX](https://sikulix.github.io/docs/scripts/python/): Visual technology to search and automate GUIs
-   [PyQt](http://www.riverbankcomputing.co.uk/software/pyqt) and [PySide](http://www.pyside.org/): Python bindings for Qt framework

### Wordlist & Password Tools
-   [PyMangle](http://code.google.com/p/pymangle/) / [py-mangle](http://code.google.com/p/pymangle/): Create word lists for penetration testing

---

# Source

- [blackhat-python3](https://github.com/EONRaider/blackhat-python3)
- [Mastering Python Scripting for System Administrators](https://github.com/PacktPublishing/Mastering-Python-Scripting-for-System-Administrators-)
- [python-pentest-tools](https://github.com/dloss/python-pentest-tools/blob/master/README.md)
