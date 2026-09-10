One of the most challenging aspects of creating a
solid trojan framework is asynchronously controlling,
updating, and receiving data from your deployed
implants. It’s crucial to have a relatively universal way
to push code to your remote trojans. This flexibility
is required not just to control your trojans in order to perform different
tasks, but also because you might have additional code that’s specific to the
target operating system.
So while hackers have had lots of creative means of command and control over the years, such as IRC or even Twitter, we’ll try a service actually
designed for code. We’ll use GitHub as a way to store implant configuration
information and exfiltrated data, as well as any modules that the implant
needs in order to execute tasks. We’ll also explore how to hack Python’s
native library import mechanism so that as you create new trojan modules,
your implants can automatically attempt to retrieve them and any dependent libraries directly from your repo, too. Keep in mind that your traffic to
GitHub will be encrypted over SSL, and there are very few enterprises that
I’ve seen that actively block GitHub itself.

one thing to note is that we’ll use a public repo to perform this testing;
if you’d like to spend the money, you can get a private repo so that prying
eyes can’t see what you’re doing. Additionally, all of your modules, configuration, and data can be encrypted using public/private key pairs, which I
demonstrate in Chapter 9. Let’s get started!
Setting Up a GitHub Account
If you don’t have a GitHub account, then head over to GitHub.com, sign up,
and create a new repository called chapter7. Next, you’ll want to install the
Python GitHub API library1 so that you can automate your interaction with
your repo. You can do this from the command line by doing the following:

> pip install github3.py

If you haven’t done so already, install the git client. I do my development from a Linux machine, but it works on any platform. Now let’s create a
basic structure for our repo. Do the following on the command line, adapting as necessary if you’re on Windows
```bash
$ mkdir trojan
$ cd trojan
$ git init
$ mkdir modules
$ mkdir config
$ mkdir data
$ touch modules/.gitignore
$ touch config/.gitignore
$ touch data/.gitignore
$ git add .
$ git commit -m "Adding repo structure for trojan."
$ git remote add origin https://github.com/<yourusername>/chapter7.git
$ git push origin master
```
Here, we’ve created the initial structure for our repo. The config directory holds configuration files that will be uniquely identified for each trojan. As you deploy trojans, you want each one to perform different tasks and
each trojan will check out its unique configuration file. The modules directory contains any modular code that you want the trojan to pick up and
then execute. We will implement a special import hack to allow our trojan
to import libraries directly from our GitHub repo. This remote load capability will also allow you to stash third-party libraries in GitHub so you don’t
have to continually recompile your trojan every time you want to add new
functionality or dependencies. The data directory is where the trojan will
check in any collected data, keystrokes, screenshots, and so forth. Now let’s
create some simple modules and an example configuration file.

 The repo where this library is hosted is here: https://github.com/copitux/python-github3/.
