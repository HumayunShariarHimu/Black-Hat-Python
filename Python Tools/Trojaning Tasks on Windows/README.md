**When you deploy a trojan, you want to perform a few
common tasks: grab keystrokes, take screenshots, and
execute shellcode to provide an interactive session to
tools like CANVAS or Metasploit. We’ll wrap things up with some sandbox detection techniques to determine if we are running within an antivirus or forensics sandbox. These
modules will be easy to modify and will work within our trojan framework.
We’ll explore man-in-the-browser-style attacks and privilege escalation techniques that you can deploy with your trojan. Each technique comes with its own challenges and probability of being caught by the
end user or an antivirus solution.**


### Notes for the reader

#### keylogger.py
This script requires `pyHook` library, that is not compatible with newer versions of Python. 

Check for details (and, in the case, updated answers on <a href="https://stackoverflow.com/questions/59968523/unable-to-install-pyhook-python-3-8-1">Stack Overflow - pyHook issues.</a>)<br>

You have two ways ahead:
- Search for a wheel file appropriate for the Python version you are running here https://www.lfd.uci.edu/~gohlke/pythonlibs/#pyhook 
- Use <a href="https://libraries.io/pypi/pyWinhook">pyWinhook</a>, a pyHook fork with some updates for support latest Visual Studio compilers.<br> 
Two methods here:<br>
____ a) `pip install pyWinhook==1.6.2`<br>
____ b) `python -m pip install pyWinhook-1.6.2-cp38-cp38-win_amd64.whl`<br><br>

I would like to switch to `keyboard` module in order to bypass all this mess, but at the moment I am not able to reproduce the exact workflow offered by the book without going thru pyHook.<br><br>

ps. if the topic is of your interest, check my keylogger for Power Shell here: https://gist.github.com/carloocchiena/316234e45f67ca63f07620e15e7dfaef

#### shell_exec.py
Be carefull to base64 encode your raw shellcode script before hosting it on localhost<br>
Store the raw shellcode in /tmp/shellcode.raw<br>
then run (on your linux shell):<br>
`user$ base64 -i shellcode.raw >`<br>
`shellcode.bin`<br>
`user$ python -m http.server 8000`<br>

#### screenshotter.py
- the win32 import issues solved thanks to this answer: https://stackoverflow.com/questions/44063350/python-no-module-named-win32gui-after-installing-pywin32?rq=1



- The bhservice folder contains material provided by the book for testing purpose. At the moment of writing the link provided by the author was not working but I managed to find it online.
- `file_monitor.py` was, accordingly to book notes, inspired by http://timgolden.me.uk/python/win32_how_do_i/watch_directory_for_changes.html
- `PyMI` provides a Python native module wrapper over the Windows Management Infrastructure (MI) API. It includes also a drop-in replacement for the Python WMI module used in the book, proving much faster execution times and no dependency on pywin32. Get it with `pip install PyMI`.
- Had some issues with `import wmi`. You may check here: https://stackoverflow.com/questions/20654047/cant-import-wmi-python-module
- Testing for this chapter is not finished. I am surely able to grab the process in `file_monitor.py` but had not yet successfully tested an injection. 
- Let me say that, after going thru 90% of the book, the author offer just zero advice on even most common issues and troubleshooting. 
