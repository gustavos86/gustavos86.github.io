---
title: Python pdb debugger notes
date: 2024-07-04 10:50:00 -0700
categories: [PYTHON, PDB]
tags: [python, python pdb]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/python_logo.png)

## Execute PDB

Use one of these three approaches to execute the pdb debugger prompt.

Include this as an additional line in the code where you wish to start the pdb debugger

```
import pdb; pdb.set_trace()
```

Starting from Python **ver 3.7**

```
breakpoint()
```

Start the Python script with:

```
python -m pdb main.py
```

## PDB Notes

- Exit pdb debugger by calling exit() OR CTRL+D (same as the Python REPL).
- If the Python code has a variable name such as l or n, then the pdb command takes precedence.
- The bang command (!) lets pdb know that the following statement will be a Python command and not a pdb command.

## PDB Commands

```
h(elp) - Without argument, print the list of available commands. With a command as an argument, print help about that command.

(Pdb) help

Documented commands (type help <topic>):
========================================
EOF    c          d        h         list      q        rv       undisplay
a      cl         debug    help      ll        quit     s        unt
alias  clear      disable  ignore    longlist  r        source   until
args   commands   display  interact  n         restart  step     up
b      condition  down     j         next      return   tbreak   w
break  cont       enable   jump      p         retval   u        whatis
bt     continue   exit     l         pp        run      unalias  where

Miscellaneous help topics:
==========================
exec  pdb

(Pdb)
```

```
l(ist) - Displays 11 lines around the current line or continue the previous listing.
ll(onglist) - List the whole source code for the current function or frame.
```

```
j(ump) lineno - Set the next line that will be executed. 
```

```
s(tep) - Execute the current line, stop at the first possible occasion.
n(ext) - Continue execution until the next line in the current function is reached or it returns.
```

```
b(reak) - Set a breakpoint (depending on the argument provided).
c(ontinue) - Executes the program and only stops if it encounters a break point.
cl(ear) - With a space separated list of breakpoint numbers, clear those breakpoints.
```

```
r(eturn) - Continue execution until the current function returns.
```

```
commands [bpnumber]
        (com) ...
        (com) end
        (Pdb)

        Specify a list of commands for breakpoint number bpnumber.
commands will run python code or pdb commands that you specified whenever the stated breakpoint number is hit. Once you start the commands block, the prompt changes to (com). The code/commands you write here function as if you had typed them at the (Pdb) prompt after getting to that breakpoint. Writing end will terminate the command and the prompt changes back to (Pdb) from (com). 
```

```
pdb.post_mortem(traceback=None)
    Enter post-mortem debugging of the given traceback object. If no traceback is given, it uses the one of the exception that is currently being handled
    (an exception must be being handled if the default is to be used).


pdb.pm()
    Enter post-mortem debugging of the traceback found in sys.last_traceback.
```

You can start the main.py script using python -m pdb main.py and continue until an exception is thrown.
Python will automatically enter post_mortem mode at the uncaught exception.

## References:
- https://github.com/spiside/pdb-tutorial
- https://realpython.com/python-debugging-pdb/