---
layout: default
permalink: /pa2/debug-notes
title:  Debug Notes
description: A compilation of debug tips for PA2
parent: Programming Assignment 2
grand_parent: Programming Assignment
nav_order:  10
---


* TOC
{:toc}

**50.005 Computer System Engineering**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design
<br>
**Natalie Agus (Summer 2024)**


# Debug Notes

### Pip Install Error

If you use Python >3.12, you might see the following error when trying to run `pip install -m requirements.txt`:

```sh
AttributeError: module 'pkgutil' has no attribute 'ImpImporter'. Did you mean: 'zipimporter'?
```

Simply upgrade pip:

```sh
python -m ensurepip --upgrade
```

### Invalid Syntax

Some of you might encounter the error when running `python3 ServerWithoutSecurity.py`

```python
  match convert_bytes_to_int(read_bytes(client_socket, 8)):
        ^
      case 0:

SyntaxError: invalid syntax
```

That's because your `python3` <span style="color:#f7007f;"><b>is NOT aliased</b></span> to `python3.10` or that you don't have `python3.10` installed. <span style="color:#f7007f;"><b>Fix this on your own.</b></span> You're a CS major student. Not knowing how to install Python and manage its libraries is _a really really bad thing_; it's like as if the entire 50.002 and the first 6 weeks of CSE doesn't mean anything to you.

In this handout, we assume that `python3` is <span style="color:#f77729;"><b>always aliased</b></span> to `python3.10`.
{:.important}

That is, if you type `python3` in the terminal, you'll see at least version 3.10 printed out:
<img src="{{ site.baseurl }}/assets/images/pa2/5.png"  class="center_full no-invert"/>

The <span style="color:#f7007f;"><b>autograder</b></span> file also uses `python3` instead of `python3.10`. You can modify it to call `python3.10` instead accordingly.

### Module Not Found

Some of you might encouter the error `ModuleNotFoundError: No module named ‘cryptography’`. You should know what you need to do by now as a CS student. If you have installed cryptography using `pip install cryptography`, but still suffer from this error, it simply means that the `pip` you used does not install to the path library of whatever `python3` version you are using right now. That is, you may have <span style="color:#f7007f;"><b>mixed</b></span> up `Python` and `pip` versions on your machine.

Assuming your `python3` is aliased to `python3.10`, then you can do:

```
python3 -m pip install cryptography
```

### No Such File or Directory: recv_files/EXPECTED_FILE

Depending on your OS, when you run `ServerWithSecurity[version].py` or the <span style="color:#f77729;"><b>autograder</b></span>, it might complain about files in the directories: `recv_files/EXPECTED_FILE`, `recv_files_enc/EXPECTED_FILE`, etc <span style="color:#f77729;"><b>not found</b></span> when you clearly have it in your project path or have run `cleanup.sh`.

This is due to differences in what constitutes a _line break_:

- `\r\n` is a Windows Style
- `\n` is a POSIX Style
- `\r` is a old pre-OS X Macs Style, Modern Mac's using POSIX Style

Therefore your directory name might be set as `recv_files\r\n` when you run `cleanup.sh`. You can't tell whether the line break is there or not. Similarly, when you receive `filename` from `input` in `ClientWithSecurity[version].py`, the line break might be there, creating filename like `files.txt\r\n`. In order to tackle this, you can:

1. Add `.strip()` at the end of `input`, resulting in `input("Enter a filename....").strip()`
2. Create a new `cleanup.sh` with the exact same content and overwrites the old one so that the line break suits your system's

When submitting, please use `LF` as your end of line sequence because we are running your code on Linux-distro containers.
{:.warning}

### Command truncate Not Found

It is possible that you might not have `truncate` system program installed by default. You have learned the File System in Week 6, so we suppose you know why we use it in our autograder.

To install, simply type `sudo apt -y install coreutils` (Ubuntu), or `brew install coreutils` (macOS).



<!-- ### Bot: Autograder Fails to Run

If the bot complains that the autograder **fails to run**, then check if you have modified the `./cleanup.sh` to be **executable**. We did not explicitly tell you to `chmod +x ./cleanup.sh` , but we _did_ ask you to run it. 

{:.highlight}
You did go through Lab 1 right? Yes? We assume you know how to _apply knowledge_.

### Local Autograder Works, but not Bot's

Check that your local autograder did printout **exactly** the following:

<img src="{{ site.baseurl }}/assets/images/pa2/3.png"  class="center_fifty no-invert"/>

That means: **no error messages** like "file A not found". We only give `autograde.py` to you so that you can have some kind of checker beforehand, but we can't guarantee 100% that it will not give false positives because you might not be using the same exact Python >3.10 version, have different system programs or different Linux distros altogether. 

If you're not sure what went wrong, **just OPEN** `autograde.py` and figure it out. It's in Python! How hard can it be? -->


## Why not give Docker Image?

{:.info-title}
> Docker?
> 
> We would've given you a Docker image but then if you can't debug your own code given the autograder written in Python, then apologies, we don't have enough faith yet that you can install Docker on your own and spawn a container properly. Maybe someday 🥹.