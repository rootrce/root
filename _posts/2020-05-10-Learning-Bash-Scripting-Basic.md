---
layout: single
title: "Basic syntax for Bash Scripting"
categories: linux
toc: true
toc_sticky: true
classes: wide
---

In this post i will keep some note about basic of bash scripting. 

**note:** File extension is not important. Saving the file as a project name is enough and giving execute permission is all we need to do.

## Hello World
```bash
#!/bin/bash
echo "Hello, We execute remote command!"
```

## Variable
```bash
#!/bin/bash
REMOTE_COMMAND="Later we will see how to execute command!"
echo $REMOTE_COMMAND
echo "What we will see? Ans: ${REMOTE_COMMAND}"
```

## Command Substitution and If statement

```bash
#!/bin/bash
USER=${UID} #executing command called substitution
ANOTHER_COMMAND=$(id -un $USER)
echo "You are ${ANOTHER_COMMAND}"
if [[ $USER -ne 0 ]] #if statement enclosed this way!
then
    echo "Not a root user"
else 
    echo "You are root!"
fi
```
**same thing in the terminal**:
```bash
pentest@kali$ if [[ $USER -ne 0 ]];
> then
> echo "You are not root"
> else
>  echo "You are root"
> fi
You are not root
```
**OR in One-line! If we press up-arrow key, we get this:**
```bash
if [[ $USER -ne 0 ]]; then echo "You are not root"; else  echo "You are root";fi
```

**Here is the other methods to compare with if condition:**

``` 
      -a FILE        True if file exists.
      -b FILE        True if file is block special.
      -c FILE        True if file is character special.
      -d FILE        True if file is a directory.
      -e FILE        True if file exists.
      -f FILE        True if file exists and is a regular file.
      -g FILE        True if file is set-group-id.
      -h FILE        True if file is a symbolic link.
      -L FILE        True if file is a symbolic link.
      -k FILE        True if file has its `sticky' bit set.
      -p FILE        True if file is a named pipe.
      -r FILE        True if file is readable by you.
      -s FILE        True if file exists and is not empty.
      -S FILE        True if file is a socket.
      -t FD          True if FD is opened on a terminal.
      -u FILE        True if the file is set-user-id.
      -w FILE        True if the file is writable by you.
      -x FILE        True if the file is executable by you.
      -O FILE        True if the file is effectively owned by you.
      -G FILE        True if the file is effectively owned by your group.
      -N FILE        True if the file has been modified since it was last read.
    
      FILE1 -nt FILE2  True if file1 is newer than file2 (according to
                       modification date).
    
      FILE1 -ot FILE2  True if file1 is older than file2.
    
      FILE1 -ef FILE2  True if file1 is a hard link to file2.
    
    String operators:
    
      -z STRING      True if string is empty.
    
      -n STRING
         STRING      True if string is not empty.
    
      STRING1 = STRING2
                     True if the strings are equal.
      STRING1 != STRING2
                     True if the strings are not equal.
      STRING1 < STRING2
                     True if STRING1 sorts before STRING2 lexicographically.
      STRING1 > STRING2
                     True if STRING1 sorts after STRING2 lexicographically.
    
    Other operators:
    
      -o OPTION      True if the shell option OPTION is enabled.
      -v VAR         True if the shell variable VAR is set.
      -R VAR         True if the shell variable VAR is set and is a name
                     reference.
      ! EXPR         True if expr is false.
      EXPR1 -a EXPR2 True if both expr1 AND expr2 are true.
      EXPR1 -o EXPR2 True if either expr1 OR expr2 is true.
    
      arg1 OP arg2   Arithmetic tests.  OP is one of -eq, -ne,
                     -lt, -le, -gt, or -ge.
    
    Arithmetic binary operators return true if ARG1 is equal, not-equal,
    less-than, less-than-or-equal, greater-than, or greater-than-or-equal
    than ARG2.
    
    Exit Status:
    Returns success if EXPR evaluates to true; fails if EXPR evaluates to
    false or an invalid argument is given.
```

**More Example**

```bash
COMMAND=$(NO Command)
if [[ ${?} -eq 0 ]] #?=last exit code 1 or 0. 0 true, 1 false
then
    echo "Command Executed"
else
    echo "Command Failed"
fi 
```

## For Loop
```bash
for files in $(ls); 
do  
    echo $files; 
done
```

## I/O Redirection
```bash
cat /etc/passwd|grep '/bin/bash'>output #Save to a file
cat <output #Taking file input
passwd --stdin username<password #taking string input

# File Descriptor code:
# 0 = STDIN
read P 0</etc/passwd
echo $P
# 1 = STDOUT
cat /etc/passwd 1>output #Only save the output
# 2 = STDERR
cat /etc/nothing 2>save_err #only save error message
head -n1 /etc/passwdd 2>&1|cat -n #&1 is special for taking the error statndard output, same as below:
head -n1 /etc/passwdd |& cat -n 
```

## Function
As bash file
```bash
#!/bin/bash
mycat(){
    echo "Hi rootrce"
    #more code....
}

#call the function
mycat
```

In terminal:
```bash
mycat(){ echo "Hi"; };mycat
```

I think this is enough to remember the bash syntax!!! In another post i will write some important Linux command usage(cut,awk,sort,uniq,head,sed etc)!