---
title: Disabling IPython AutoComplete
date: 2024-07-31 21:00:00 -0700
categories: [PYTHON IPYTHON]
tags: [python]     # TAG names should always be lowercase
---

I use **IPython** quite often to validate my code and ensure that it is going to work the way I intended. There is something I don't like from **IPython** though and that is its autocomplete feature.

Most of the time, it tries to autocomplete what I am writing with code that I do not need. See the next screenshot

![]({{ site.baseurl }}/images/2024/07-31-Disabling-IPython-AutoComplete/01-IPython-autocomplete-example.png)

I just learned this feature can be disabled, so not only I am doing that but also documenting how to do it for future references.

## On Windows

On a **Terminal** or **Command Prompt**

```
ipython profile create
```

<details markdown=1>
<summary markdown="span">output</summary>

```
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Users\user> ipython profile create
[ProfileCreate] Generating default config file: WindowsPath('C:/Users/user/.ipython/profile_default/ipython_config.py')
PS C:\Users\user>
```
</details><br />

Now, open the file with a text edit. Notepad would work just fine

```
notepad.exe <FILE>
```

<details markdown=1>
<summary markdown="span">output</summary>

```
PS C:\Users\user> notepad.exe C:/Users/user/.ipython/profile_default/ipython_config.py
PS C:\Users\user>
```
</details><br />

Ensure the following configuration lines are in the file

```python
c = get_config()
c.TerminalInteractiveShell.autosuggestions_provider = None
```

Example:

![]({{ site.baseurl }}/images/2024/07-31-Disabling-IPython-AutoComplete/02-IPython-config-file-on-Windows.png)

The next time **IPython** is opened, autocomplete will be disabled

![]({{ site.baseurl }}/images/2024/07-31-Disabling-IPython-AutoComplete/03-IPython-disabled-on-Windows.png)

## On Mac OS X and Linux

The process is quite similar on **Mac OS X** and **Linux**. If the file is not there yet, create it in the following path using the text edit of your preference (I use [Vim](https://www.vim.org/))

```bash
~/.ipython/profile_default/ipython_config.py
```

The file should contain the following configuration lines which are actually the same ones than in Windows.

```python
c = get_config()
c.TerminalInteractiveShell.autosuggestions_provider = None
```

Done, next time **IPython** is opened, autocomplete will be disabled

## References

- [IPython Website](https://ipython.org/)