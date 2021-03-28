---
title: centos7
tags: 
  - centos
  - shell script
  - Linux
date: 2016-08-29 16:18:03
---
### CentOS quck start guide

## Analyzing syslog entries
![log file standard format](https://i.imgur.com/X4FxDgT.png)





### 1. Stop and Start RHEL7 firewall
```
[root@rhel7 ~]# service firewalld stop
Redirecting to /bin/systemctl stop  firewalld.service

[root@rhel7 ~]# service firewalld start
Redirecting to /bin/systemctl start  firewalld.service
```


### 2. Disable and Enable RHEL7 firewall
```
[root@rhel7 ~]# systemctl disable firewalld
rm '/etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service'
rm '/etc/systemd/system/basic.target.wants/firewalld.service'


[root@rhel7 ~]# systemctl enable firewalld
ln -s '/usr/lib/systemd/system/firewalld.service' '/etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service'
ln -s '/usr/lib/systemd/system/firewalld.service' '/etc/systemd/system/basic.target.wants/firewalld.service'
```

### 3. Set hostname
```
[root@rhel7 ~]# hostnamectl

[root@rhel7 ~]# hostnamectl set-hostname nameyoulike
```

# Signals
```
[root@ip-172-31-5-191 ~]# kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX
```
## SIGKILL signal
```
[root@ip-172-31-5-191 ~]# sleep 10000 &
[1] 19559
[root@ip-172-31-5-191 ~]# ps aux|grep sleep
root     19559  0.0  0.0 107952   356 pts/0    S    07:48   0:00 sleep 10000
root     19564  0.0  0.0 112708   976 pts/0    S+   07:48   0:00 grep --color=auto sleep
[root@ip-172-31-5-191 ~]# kill -9 19559
[1]+  Killed                  sleep 10000
[root@ip-172-31-5-191 ~]# ps aux|grep sleep
root     19596  0.0  0.0 112708   980 pts/0    S+   07:49   0:00 grep --color=auto sleep
```

## SIGHUP signal
```
[root@ip-172-31-5-191 ~]# nohup sleep 1000 &
[1] 19786
[root@ip-172-31-5-191 ~]# nohup: ignoring input and appending output to ‘nohup.out’
```

## Bash Shell Variables

Put variable into the file `/etc/environment`

Another very important rule is that a child process will never be able to change the parent's environment variables, because the child and parent are independent from each other and the child only has a local copy of the parent's environment:

```
[root@ip-172-31-5-191 ~]# mkdir ~/scripts
[root@ip-172-31-5-191 ~]# printf '#!/bin/bash\nexport CHILDVAR=Hello_from_child\n' > ~/scripts/child.sh
[root@ip-172-31-5-191 ~]# cat ~/scripts/child.sh
#!/bin/bash
export CHILDVAR=Hello_from_child
[root@ip-172-31-5-191 ~]# chmod +x ~/scripts/child.sh
[root@ip-172-31-5-191 ~]# ~/scripts/child.sh
[root@ip-172-31-5-191 ~]# echo $CHILDVAR
```

## Bash Shell Scripting
### Functions
```
[root@ip-172-31-5-191 ~]# say_hello() {
> echo "My name is $1";
> }

[root@ip-172-31-5-191 ~]# say_hello Stan
My name is Stan
[root@ip-172-31-5-191 ~]# say_another_thing() {
> say_hello Stan
> echo "I like CentOS 7";
> }

[root@ip-172-31-5-191 ~]# say_another_thing
My name is Stan
I like CentOS 7
```
---
```
[root@ip-172-31-5-191 ~]# cat run.sh
#!/bin/bash
echo "Hello world"
echo "-----------"
if [[ $# -lt 2 ]]
then
  echo "Usage $0 param1 param2"
  exit 1
fi
echo $1
echo $2
echo $0
echo $#
```
---
```
[root@ip-172-31-5-191 ~]# PASSWORD="tt4321"
[root@ip-172-31-5-191 ~]# if [[ $PASSWORD -eq "hello_world" || $PASSWORD -eq "tt4321" ]]; then echo "password correct"; fi
password correct
```
### if
```
if [[ $EXIT -eq 0 ]]
if ! [[ $EXIT -eq 0 ]]
if [[ $PASSWORD == "Hello_world" ]]
if [[ $PASSWORD -eq "hello_world" || $PASSWORD -eq "tt4321" ]]

STRING="Lorem ipsum dolor sit"
if [[ $STRING =~ ^..rem ]]
if [[ $NUMBER -lt 10 ]]  ##-lt less than
if [[ $NUMBER -gt 10 ]]  ##-gt greater than
```
### file or diri tests
```
if [[ -a $FILE ]]
if [[ -d $DIR ]]
```
**for read the bash manual**
`man bash` /CONDITIONAL EXPRESSIONS

### else statement
```
if [[ $EXIT -eq 0 ]]
then
  echo "whatever"
else
  echo "cannot access the files"
fi
```
### if elif statement

```
if [[ $VALUE -gt 5 ]]
then
  echo "value is bigger than 5"
elif [[ $VALUE -eq 5 ]]
then 
  echo "value is equal to 5"
elif [[ $VALUE -lt 5 ]]
then
  echo "value is less than 5"
fi
```
### Loops statement
```
for count in 1 2 3 4
do
  echo $count
done

for number in $(seq 1 20)
do
 echo "This is $number"
done

for number in {1..20}
do
 echo "This is $number"
done
```
change multi file names
```
mkdir ~/stuff
touch ~/stuff/1.txt ~/stuff/2.txt ~/stuff/3.txt ~/stuff/4.txt ~/stuff/5.txt
for file in ~/stuff/*.txt
do
  mv $file ~/stuff/$(basename $file .txt).doc
done
```
### Automate script execution
```
cat scripts/motd-commandlinefu-update.sh
#!/bin/bash
wget -O /etc/motd http://www.commandlinefu.com/commands/random/plaintext
ls /etc/cron* -d
/etc/cron.d  /etc/cron.daily  /etc/cron.hourly  /etc/cron.monthly  /etc/crontab  /etc/cron.weekly
root@stan-OptiPlex-380:~|⇒  cd /etc/cron.daily
root@stan-OptiPlex-380:/etc/cron.daily|
⇒  ln -s /root/scripts/motd-commandlinefu-update.sh
root@stan-OptiPlex-380:/etc/cron.daily|
⇒  ll
total 72
drwxr-xr-x   2 root root  4096 Jun 20 10:47 ./
drwxr-xr-x 150 root root 12288 Jun 20 10:44 ../
-rwxr-xr-x   1 root root   311 May 30  2017 0anacron*
-rwxr-xr-x   1 root root   539 Oct 11  2018 apache2*
-rwxr-xr-x   1 root root   376 Nov 21  2017 apport*
-rwxr-xr-x   1 root root  1478 Apr 20  2018 apt-compat*
-rwxr-xr-x   1 root root   314 Jan 17  2018 aptitude*
-rwxr-xr-x   1 root root   355 Dec 29  2017 bsdmainutils*
-rwxr-xr-x   1 root root   384 Dec 13  2012 cracklib-runtime*
-rwxr-xr-x   1 root root  1176 Nov  3  2017 dpkg*
-rwxr-xr-x   1 root root   372 Aug 22  2017 logrotate*
-rwxr-xr-x   1 root root  1065 Apr  7  2018 man-db*
-rwxr-xr-x   1 root root   538 Mar  2  2018 mlocate*
lrwxrwxrwx   1 root root    42 Jun 20 10:47 motd-commandlinefu-update.sh -> /root/scripts/motd-commandlinefu-update.sh*
-rwxr-xr-x   1 root root   249 Jan 26  2018 passwd*
-rw-r--r--   1 root root   102 Nov 16  2017 .placeholder
-rwxr-xr-x   1 root root   246 Mar 22  2018 ubuntu-advantage-tools*
```
---
## Working in the shell efficiently
man bash /Commands for Moving

### Command Editing Shortcuts
- `Ctrl + a` – go to the start of the command line
- `Ctrl + e` – go to the end of the command line
- `Ctrl + k` – delete from cursor to the end of the command line
- `Ctrl + u` – delete from cursor to the start of the command line
- `Ctrl + w` – delete from cursor to start of word (i.e. delete backwards one word)
- `Ctrl + y` – paste word or text that was cut using one of the deletion shortcuts (such as the one above) after the cursor
- `Ctrl + xx` – move between start of command line and current cursor position (and back again)
- `Alt + b` – move backward one word (or go to start of word the cursor is currently on)
- `Alt + f` – move forward one word (or go to end of word the cursor is currently on)
- `Alt + d` – delete to end of word starting at cursor (whole word if cursor is at the beginning of word)
- `Alt + c` – capitalize to end of word starting at cursor (whole word if cursor is at the beginning of word)
- `Alt + u` – make uppercase from cursor to end of word
- `Alt + l` – make lowercase from cursor to end of word
- `Alt + t` – swap current word with previous
- `Alt + .` – print previous command's argument
- `Ctrl + f` – move forward one character
- `Ctrl + b` – move backward one character
- `Ctrl + d` – delete character under the cursor
- `Ctrl + h` – delete character before the cursor
- `Ctrl + t` – swap character under cursor with the previous one
### Command Recall Shortcuts
- `Ctrl + r` – search the history backwards
- `Ctrl + g` – escape from history searching mode
- `Ctrl + p` – previous command in history (i.e. walk back through the command history)
- `Ctrl + n` – next command in history (i.e. walk forward through the command history)
- `Alt + .` – use the last word of the previous command
### Command Control Shortcuts
- `Ctrl + l` – clear the screen
- `Ctrl + s` – stops the output to the screen (for long running verbose command)
- `Ctrl + q` – allow output to the screen (if previously stopped using command above)
- `Ctrl + c` – terminate the command
- `Ctrl + z` – suspend/stop the command
### for debuging purpose
- `Ctrl + Alt + e` - $SHELL /bin/bash
### Bash Bang (!) Commands
Bash also has some handy features that use the ! (bang) to allow you to do some funky stuff with bash commands.

- `!!` – run last command
- `!blah` – run the most recent command that starts with ‘blah’ (e.g. !ls)
- `!blah:p` – print out the command that !blah would run (also adds it as the latest command in the command history)
- `!$` – the last word of the previous command (same as Alt + .)
- `!$:p` – print out the word that !$ would substitute
- `!*` – the previous command except for the last word (e.g. if you type ‘_find somefile.txt /’, then !* would give you ‘_find somefile.txt’)
- `!*:p` – print out what !* would substitute
- `tail -f log_file | egrep -v 'ELB|Pingdom|Health'` – filter out stuff has certain keywords
