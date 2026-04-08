# Linux - System Administration
## Chapter 1: Fundamentals

---

## Introduction

**Linux** is widely distributed in the enterprise world. **Linux administration** involves a wide variety of tasks, from running web servers to configuring systems.

Fields like **networking**, **game development**, **AI**, and **big data** are rooted in Linux.

---

## The Kernel

**Linux** refers to the **kernel**, the brain of a system.

Some people refer to "Linux" as a collection of free software **GNU/Linux** (like [Ubuntu](https://ubuntu.com/)), but behind it all, it's still the **Linux kernel** running at boot time that manages the system.

### UNIX History

Linux started with **UNIX**, an operating system developed by **AT&T Bell Labs** in the **1970s**, written in **C language** (a low-level language).

Right now, Unix is a **trademark** and a specification owned by [**The Open Group**](https://www.opengroup.org/).

### Birth of Linux

Over time, Unix was forked. In **1991**, [**Linus Torvalds**](https://github.com/torvalds) created a **UNIX-like OS** for educational use.

Despite adopting all the requirements of the UNIX specification, it has not been certified, so **Linux isn't UNIX**, just **UNIX-like**.

### The GNU Project

Alongside, [**Richard Stallman**](https://stallman.org/) in **1983** created the [**GNU Project**](https://www.gnu.org/). While trying to build their own OS, they realized they were better at building **tools** that match UNIX-like OS.

> ### 💡 **Consider This**
> 
> **Linus** originally named the project **Freax**, however, an administrator of the server where the development files were uploaded renamed it **Linux**, a portmanteau of Linus' name and UNIX. The name stuck.
> 
> **GNU** is a recursive acronym for **"GNU's Not Unix,"** and it's pronounced just like the African horned antelope that is its namesake.

| Skill | Score |
|-------|:-----:|
| Describe the role of the Linux kernel | 5 |
| Explain the difference between Linux and GNU/Linux | 5 |
| Describe the history of UNIX and its relation to Linux | 5 |
| Explain why Linux is UNIX-like but not UNIX certified | 4 |

---

## Open Source

Most software has been issued with a **closed source license**: you can use the product but you don't have access to the **source code**, and sometimes you're required not to try to **reverse engineer** the source code.

The development of Linux follows the same path as **Open Source**: you have the right to obtain the software **source code** and **modify** it.

It was not the first software doing so, but since it was built **from scratch**, early adopters could **influence the project greatly**, accelerating the pace of development and preventing mistakes from other OS.

> ### 💡 **Consider This**
> 
> The source code may be written in any of hundreds of different languages. Linux happens to be written in **C**, a **versatile** and relatively **easy language to learn**, which shares history with the original UNIX. This decision, made long before its utility was proven, turned out to be **crucial** in its nearly universal adoption as the primary operating system for **internet servers**.

| Skill | Score |
|-------|:-----:|
| Explain what open source software is | 5 |
| Differentiate open source vs closed source licensing | 5 |
| Describe common open source licenses (GPL, MIT, Apache) | 3 |
| Understand the concept of copyleft | 3 |

---

## Linux Distributions

A **distribution** is a set of tools and suite of applications that come bundled together.

Like Linux and the **GNU tools** that add web browser and email client, some individuals and companies create their own distribution that takes care of installing the kernel, for example.

But they also include a **package manager** to help you manage software on your machine.

### Types of Distributions

Some distributions focus on:
- **Running servers**
- **Desktops**
- **Specialized usage**

### Major Players

The **major players** are:
- [**Red Hat**](https://www.redhat.com/) (RHEL, Fedora, CentOS Stream)
- [**Debian**](https://www.debian.org/) (Ubuntu, Linux Mint)
- [**SUSE**](https://www.suse.com/) (openSUSE)
- [**Slackware**](http://www.slackware.com/)

The visible differences can range from **file locations** to **political philosophies**.

| Skill | Score |
|-------|:-----:|
| Identify major Linux distributions and their families | 5 |
| Explain the role of a package manager | 4 |
| Describe use cases for different distributions | 4 |
| Differentiate desktop, server, and embedded distributions | 4 |

---

## Command Line Interface (CLI)

There are **two ways** of interacting with the computer:
1. **GUI** (**Graphical User Interface**)
2. **CLI** (**Command Line Interface**) - a text-based interface

### CLI Advantages

Traditionally, OS offer both **GUI** and **CLI** interfaces. The **CLI** provides:
- ✅ **More precise control**
- ✅ **Greater speed**
- ✅ **Ability to automate tasks** through **scripting**

| Skill | Score |
|-------|:-----:|
| Describe the difference between GUI and CLI | 5 |
| Explain the advantages of CLI over GUI | 5 |
| Understand what a shell is and its role | 5 |
| Use basic shell commands (ls, cd, pwd, echo) | 5 |

---

## 📚 References — LPI Linux Essentials (010-160)

**Topic 1: The Linux Community and a Career in Open Source**
- **1.1** Linux Evolution and Popular Operating Systems
- **1.3** Understanding Open Source Software and Licensing
- **1.4** ICT Skills and Working in Linux

### Official Links:
- [Linux Professional Institute (LPI)](https://www.lpi.org/)
- [Linux Essentials Exam Objectives](https://www.lpi.org/our-certifications/linux-essentials-overview/)
- [The Linux Kernel Archives](https://kernel.org/)
- [GNU Operating System](https://www.gnu.org/)

---

### 🔑 **Important Keywords:**
`kernel`, `GNU/Linux`, `open source`, `distributions`, `CLI`, `GUI`, `UNIX-like`, `C language`, `package manager`, `scripting`, `closed source license`, `reverse engineer`, `copyleft`, `GPL`
