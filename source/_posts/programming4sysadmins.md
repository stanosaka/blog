---
title: programming4sysadmins
date: 2019-07-19 16:10:53
tags: [programming, systemadmin, python]
---
# python
## What can you do with python?
- Versatile language, can use it across a lot of different domains
- really fast to learn and fast to develop in
## Environment Setup
[link](https://wiki.openhatch.org/wiki/O%27Reilly_Introduction_to_Python)
## Python basic data types
```
>>> type(1)
<type 'int'>
>>> type(1.0)
<type 'float'>
Type is a functions that takes input and spits out output.
```
**a function** is just the name and then input inside parentheses, and it'll spit out output.

If you need to include the string delimiter inside the string just precede it with a backslash, as in 'It\'s a wrap'
```
>>> "Hello " + str(1)
'Hello 1'
```
## Making choices: boolean, if/eif/else, compound conditionals
Boolean:
- True
- False
```
>>> x = 1
>>> x > 0 and x < 2
True
>>> "a" in "hello" or "e" in "hello"
True

>>> temp = 32
>>> if temp > 60 and temp < 75:
...     print("Nice and cozy")
... else:
...     print("Too extreme for me")
...
Too extreme for me

>>> sister = 15
>>> brother =15
>>> if sister > brother:
...     print("Sister is older")
... elif sister == brother:
...     print("Same age!")
... else:
...     print("Brother is older")
...
Same age!
```
## Lists
```
>>> names = ["Alice", "Amy"]
>>> names.append("Adam")
>>> names
['Alice', 'Amy', 'Adam']
>>> names[len(names)-1]
'Adam'
>>> names[-1]
'Adam'
```
The real superpower when using lists is actually to be able to loop over them.

## Loops

# Great Bash
- Redirect the output of commands
  - Standard error is file descriptor "2" `ls -l myscript not.here > lsout 2> lserr`
  - Out and error can be redirected separately or together `ls -l myscript not.here &> lsboth`
  - The order of redirection is important
```
ls -l myscript not.here > lsout 2>&1
## Redirectin error output to standard output
## Standard output is already being re-directed to file > dirlist
## Hence, both error and standard output are written to file lsout
```
- Redirecting and piping input and output
```
ls > /tmp/lsout
wc < /tmp/lsout
```
Use the vertical bar character | to create a pipe: `ls|wc`

Connect a series of commands with | symbols to make a pipeline.

- Create input with here documents

