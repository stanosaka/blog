---
title: python for devops
tags:
    - python
    - devops
---
# RE Package
## Character sets

The *+* after an item in a regular expression matches one or more of it. A number in brackets matches an exact number of characters.
```
In [6]: print cc_list
Ezra Koenig <ekoenig@vpwk.com>,
Rostam Batmanglij <rostam@vpwk.com>,
Chris Tomson <ctomson@vpwk.com,
Bobbi Baio <bbaio@vpwk.com

>>> re.search(r'[A-Za-z]{6}', cc_list)
<_sre.SRE_Match object; span=(5, 11), match='Koenig'>

```
The *.* character has a special meaning. It is a wildcard and matches any character. To match against the actual *.* character, you must escape it using a backslash:
```
>>> re.search(r'[A-Za-z]+@[a-z]+\.[a-z]+', cc_list)
<_sre.SRE_Match object; span=(13, 29), match='ekoenig@vpwk.com'>

```
## Character Classes
Python’s re offers character classes. These are pre-made characters sets. Some commonly used ones are \w, which is equivalent to [a-zA-Z0-9_] and \d which is equivalent to [0-9]. You can use the + modifier to match for multiple characters:
```
>>> re.search(r'\w+', cc_list)
<_sre.SRE_Match object; span=(0, 4), match='Ezra'>
>>> re.search(r'\w+\@\w+\.\w+', cc_list)
<_sre.SRE_Match object; span=(13, 29), match='ekoenig@vpwk.com'>

```
## Groups
```
>>> matched = re.search(r'(\w+)\@(\w+)\.(\w+)', cc_list)
>>> matched.group(0)
'ekoenig@vpwk.com'
>>> matched.group(1)
'ekoenig'
>>> matched.group(2)
'vpwk'
```
## Named Groups
You can also supply names for the groups by adding **?P<NAME>** in the group definition. Then you can access the groups by name instead of number:

```
>>> matched = re.search(r'(?P<name>\w+)\@(?P<SLD>\w+)\.(?P<TLD>\w+)', cc_list)
>>> matched.group('name')
'ekoenig'
>>> matched.group('SLD')
'vpwk'
```
## Using IPython to run unix shell commands
```
In [5]: ls = !ls -l /usr/bin

In [6]: ls.grep("kill")
Out[6]:
['-rwxr-xr-x 1 root root        1812 Sep 11  2018 cinnamon-killer-daemon',
 '-rwxr-xr-x 1 root root       27768 Dec 12  2018 killall',
 'lrwxrwxrwx 1 root root           5 May 14  2018 pkill -> pgrep',
 '-rwxr-xr-x 1 root root       26704 May 14  2018 skill',
 'lrwxrwxrwx 1 root root           5 May 14  2018 snice -> skill',
 '-rwxr-xr-x 1 root root       14328 Apr 22  2017 xkill']
```
**Using ipython magic commands**
```
In [7]: %%bash
   ...: uname -a
   ...:
Linux stan-OptiPlex-380 4.15.0-20-generic #21-Ubuntu SMP Tue Apr 24 06:16:15 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```
The **%%writefile** is pretty tricky because you can write and test Python or Bash scripts on the fly, using IPython to execute them.  Not a bad party trick at all:
```
In [8]: %%writefile print_time.py
   ...: #!/usr/bin/env python
   ...: import datetime
   ...: print(datetime.datetime.now().time())
   ...:
Writing print_time.py

In [9]: cat print_time.py
#!/usr/bin/env python
import datetime
print(datetime.datetime.now().time())
In [10]: !python print_time.py
16:49:14.836331
```
Another very useful command, %who, will show you what is loaded into memory.  It comes in quite handy when you have been working in a terminal that has been running for a long time:
```
In [11]: %who
ls       var_ls
```
# Package Management
## Descriptive Versioning
Most commonly in Python packages the following two variants: major.minor or major.minor.micro
- 0.0.1
- 1.0
- 2.1.1

A commonly accepted format for releases is: for **major.minor.micro** (used by the [Semantic Versioning scheme](https://semver.org/)
- major for backwards-incompatible changes
- minor adds features that are also backward-compatible
- micro adds backwards-compatible bug fxes

Assuming the current released version of the project under development is 1.0.0, it means the following outcomes are possible:
- If the release has backward-incompatible changes, the version is: 2.0.0
- Adding features that do not break compatibility: 1.1.0
- Fixing issues that also do not break compatibility: 1.0.1

## The changelog
The below example is an actual portion of a **changelog** file in a production Python tool:
```
1.1.3  
-----  
22-Mar-2019  
* No code changes - adding packaging files for Debian  

1.1.2  -----  
3-Mar-2019

* Try a few different executables (not only ``python``) to check for a working one, in order of preference, starting with ``python3`` and ultimately falling    back to the connection interpreter
```
The example has four essential items that are providing information:
1. The lastest version number released
2. Is the latest release a backward-compatible one
3. What was the release date of the last version
4. Changes included in the release

## Packaging solutions
A small Python project:
```
hello-world
── hello_world   
   ├── __init__.py   
   └── main.py

1 directory, 2 files
```
## Native Python packaging
Like the rest of the other packaging strategies, the project requires some files used by setuptools.
To continue, create a new virtual environment and then activate:
```
$ python3 -m venv /tmp/packaging
$ source /tmp/packaging/bin/activate
```
Setuptools is a requirement to produce a native Python package. It is a collection of tools and helpers to create and distribute Python packages.

Once the virtual environment is active,  the following dependencies exist:
1. setuptools A set of utilities for packaging

2. twine A tool for registering and uploading packages

```
$ pip install setuptools twine
```
**Package files**
The file that describes the package to setuptools is named setup.py. It exists at the top level directory. 
```
cat setup.py
from setuptools import setup, find_packages

setup(
    name="hello-world",
    version="0.0.1",
    author="Example Author",
    author_email="author@example.com",
    url="example.com",
    description="A hello-world example pacakge",
    packages=find_packages(),
    classifiers=[
       "Programming Language :: Python :: 3",
       "License :: OSI Approved :: MIT License",
       "Operating System :: OS Independent",
    ],
)
```
The setup.py file will import two helpers from the setuptools module: setup, and find_packages. The setup function is what requires the rich description about the package, and the find_packages function is a utility to automatically detect where the Python files are, lastly, the classifiers describes certain aspects of the package like the license, operating systems supported, and Python versions. These classifiers are called trove classifiers and the Python Package Index has a detailed description of other classifiers available. Detailed descriptions make a package get discovered when uploaded to PyPI.
```
 hello-world tree
.
├── hello-world
│   ├── __init__.py
│   └── main.py
├── README
└── setup.py
```
To produce the source distribution from it, run the following command:
```
 hello-world python3 setup.py sdist
running sdist
running egg_info
creating hello_world.egg-info
writing hello_world.egg-info/PKG-INFO
writing dependency_links to hello_world.egg-info/dependency_links.txt
writing top-level names to hello_world.egg-info/top_level.txt
writing manifest file 'hello_world.egg-info/SOURCES.txt'
reading manifest file 'hello_world.egg-info/SOURCES.txt'
writing manifest file 'hello_world.egg-info/SOURCES.txt'
running check
creating hello-world-0.0.1
creating hello-world-0.0.1/hello-world
creating hello-world-0.0.1/hello_world.egg-info
copying files to hello-world-0.0.1...
copying README -> hello-world-0.0.1
copying setup.py -> hello-world-0.0.1
copying hello-world/__init__.py -> hello-world-0.0.1/hello-world
copying hello-world/main.py -> hello-world-0.0.1/hello-world
copying hello_world.egg-info/PKG-INFO -> hello-world-0.0.1/hello_world.egg-info
copying hello_world.egg-info/SOURCES.txt -> hello-world-0.0.1/hello_world.egg-info
copying hello_world.egg-info/dependency_links.txt -> hello-world-0.0.1/hello_world.egg-info
copying hello_world.egg-info/top_level.txt -> hello-world-0.0.1/hello_world.egg-info
Writing hello-world-0.0.1/setup.cfg
creating dist
Creating tar archive
removing 'hello-world-0.0.1' (and everything under it)
hello-world tree
.
├── dist
│   └── hello-world-0.0.1.tar.gz
├── hello-world
│   ├── __init__.py
│   └── main.py
├── hello_world.egg-info
│   ├── dependency_links.txt
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   └── top_level.txt
├── README
└── setup.py

3 directories, 9 files
```
The newly created tar.gz file is an installable package! This package can now be uploaded to the Python Package Index for others to install right from it. By following the version schema, it allows installers to ask for a specific version (0.0.1 in this case), and the extra metadata passed into the setup() function enables other tools to discover it and show information about it like the author, description, and version.

```
 hello-world pip install dist/hello-world-0.0.1.tar.gz
Processing ./dist/hello-world-0.0.1.tar.gz
  Requirement already satisfied (use --upgrade to upgrade): hello-world==0.0.1 from file:///home/stan/Learning/python4devops/hello-world/dist/hello-world-0.0.1.tar.gz in /tmp/packaging/lib/python3.6/site-packages
Building wheels for collected packages: hello-world
  Running setup.py bdist_wheel for hello-world ... done
  Stored in directory: /home/stan/.cache/pip/wheels/09/67/3a/398ce8c37594dc628e00ad661230c0ed20a2f3b0f421601264
Successfully built hello-world
```
### The python package index
PyPI is a repository of Python software that allows users to host Python pakcages and also install from it.

[Register online](https://test.pypi.org/account/register/)
```
hello-world twine upload --repository-url https://test.pypi.org/legacy/ dist/hello-world-0.0.1.tar.gz
Enter your username: foobar
Enter your password:
Uploading distributions to https://test.pypi.org/legacy/
Uploading hello-world-0.0.1.tar.gz
100%|████████████████████████████████████| 3.86k/3.86k [00:01<00:00, 3.27kB/s]```
Be helpful to create a Makefile and put a make command in it automatically deploys your project and build the documentation for you.
```
deploy-pypi:
      pandoc --from=markdown --to=rst README.md -o README.rst
      python setup.py check --restructuredtext --strict --metadata
      rm -rf dist
      python setup.py sdist
      twine upload dist/*
      rm -f README.rst
```
### Hosting an internal package index
If package A , is hosted internally and has requirements on packages B and C, all those three need to exist (along with their required versions) in the same instance.

**note**
A highly recommended full-featured tool for hosting an internal PyPI is devpi. It has features like mirroring, staging, replication, and Jenkins integration. The [project documentation](http://doc.devpi.net/) has great examples and detailed information.

Now copy the tar.gz file into the hello-world directory. The final version of this directory structure should look like this:
```
(packaging) ➜  pypi tree
.
├── hello-world
│   └── hello-world-0.0.1.tar.gz
└── hello-world1
    └── hello-world1-0.0.1.tar.gz
 pypi pwd
/home/stan/Learning/python4devops/pypi

pypi python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
127.0.0.1 - - [01/Sep/2019 16:46:27] "GET /hello-world1/ HTTP/1.1" 200 -
127.0.0.1 - - [01/Sep/2019 16:46:27] "GET /hello-world1/hello-world1-0.0.1.tar.gz HTTP/1.1" 200 -

$ python3 -m venv /tmp/local-pypi
$ source /tmp/local-pypi/bin/activate
pip3 install -i http://localhost:8000/ hello-world1
Requirement already satisfied: hello-world1 in /tmp/local-pypi/lib/python3.6/site-packages

```
## Debina packaging


