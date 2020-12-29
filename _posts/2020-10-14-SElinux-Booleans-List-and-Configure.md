---
layout: post
title: Booleans - List and Configure
subtitle: SELinux Management
tags: [rhel, exam, selinux]
gh-badge: [star, fork, follow]
comments: true
---

## SELinux

## Booleans - **List and Configure**

Booleans allow some SELinux policies to be changed at runtime and persistentle, without interaction(changes) of polices.

## 1. List the Booleans

---

### a. To **list all** booleans existing on the system **with details**, use:
``` bash
#-l - listing (the same with fcontext -l, ports -l etc.) 
semanage boolean -l
```
![](https://i.imgur.com/lQbff6d.png)

On the screenshot above, I used the `head` command to shorten the output. (the first 10 lines by default). The image illustartes, 4 colomns:

1. SElinux boolean: the name of the specific boolean (bool *for short*)
> e.g. **antivirus_use_jit**
2. State: the current state of the specific boolean. *on - active, off - inactive*. Later, I will explain how to change the current state. 
> e.g. **off**
3. Deafult: the default state of the specific boolean. The default state of boolean applied to the boolean after a reboot. Later, I will explain how to change the default state. 
> e.g. **off**
4. Description: the short explanation of the specific boolean 
> e.g **Allow antivirus to use jit**

---

### b. To **show the current state** of all or specific booleans existing on the system, use:

``` bash
#-a - all
getsebool -a
```
![](https://i.imgur.com/sCv7ZdC.png)


*or*

``` bash
#type the specific boolean
getsebool httpd_can_network_connect
```
![](https://i.imgur.com/R1gzMFw.png)

> **NOTE!** `tab` can be used for input completion (bash completion)

As you see from the screenshots above, there is neither **default state**, no **description** columns. Only the current state of the booleans


## 2. Configure the Booleans (current and default states)



1. To enable or disable boolean **on runtime** (runtime means that this is only applied druing the current session, aftet a reboot, the current state will be dropped to the default state):

``` bash
#to enable (on) the boolean on runtime/
#Allow the httpd service to connect to the network.
setsebool httpd_can_network_connect on
```
``` bash
#to disable (off) the boolean on runtime.
#Prohibit the httpd service to connect to the network.
setsebool httpd_can_network_connect off
```
![](https://i.imgur.com/s6Ri9HM.png)
> REMEMBER! It does not change the DEFAULT STATE at all.

2. To enable or disable boolean **persistently** (persistently means that the current state will be copied to the default state and remain persistant):

``` bash
#to enable (on) the boolean persistently.
#Allow the httpd service to connect to the network.
setsebool -P httpd_can_network_connect on
```
``` bash
#to disable (off) the boolean persistently.
#Prohibit the httpd service to connect to the network.
setsebool -P httpd_can_network_connect off
```
Before we run those commands, let's check the default state of **httpd_can_network_connect** boolean. But because we cannot list the default state using `getsebool httpd_can_network_connect`, we definitely use `semanage boolean -l | grep httpd_can_network_connect | head -n 1`:

![](https://i.imgur.com/h3nJMCK.png)
> red arrow - current state, blue arrow - default state

As we noticed from the screenshot above, the default state is "off".
Now, let's run `setsebool -P httpd_can_network_connect on` and `setsebool -P httpd_can_network_connect off` commands I have mentioned above.
![](https://i.imgur.com/6UgfCvy.png)

![](https://i.imgur.com/qMcoREe.png)

So you see, using "-P" switch we can change the default state of boolean (change persistently).

## The alternative method of changing the default state is:

``` bash
#-m - modify the specific boolean.
#-1 - enable (--on).
#to enable (on) the boolean persistently.
#Allow the httpd service to connect to the network.
semanage boolean -m httpd_can_network_connect -1

```

``` bash
#-m - modify the specific boolean.
#-0 - disable (--off).
#to disable (off) the boolean persistently.
#Prohibit the httpd service to connect to the network.
semanage boolean -m httpd_can_network_connect -0
```
![](https://i.imgur.com/Y1q6TJc.png)

## Thank you!



