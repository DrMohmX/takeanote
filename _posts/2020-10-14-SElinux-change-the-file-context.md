---
layout: post
title: SELinux Management
subtitle: Create and Restore SELinux fcontext
tags: [rhel, exam, selinux]
gh-badge: [star, fork, follow]
comments: true
---

## SELinux

## Change the file context type (fcontext) 
We can change the fcontext **persistenty** and **temporary**. Temporary means on runtime, and it will be restored after the next reboot or running `restorecon` command. But persistantly means remain even after reboot or `restorecon` command.

### Changing the fcontext of the file/dir **temporary!**

1. Add a directory:
``` bash
mkdir /root/test
```
2. Check out the directory properties (SELinux content):
``` bash
ls -lZd /root/test
```
![](https://i.imgur.com/hZuwR02.png)
as we see the context user (**unconfined_u**), the context role (**object_r**) and context type (**admin_home_t**). For RHEL 8 exam is enough to know only about context type, the other two (user and role) are out of the scope of RHEL exam. 
> **REMEMBER!** The file/dir inherits the SELinux content of the parent dir. But if you create file/dir under the root it will take **default_t** fcontext type.

3. Now it's time to change the file context (fcontext) of our previously created directory ./test:
``` bash
# Change only the directory fcontext
#-v - verbosity
#-t - fcontext type
chcon -vt httpd_sys_content_t /root/test/
```
``` bash
# Change recursively all the files under ../test directory (-R switch)
#-v - verbosity
#-t - fcontext type
#-R - recursively
chcon -vRt httpd_sys_content_t /root/test/
```

![](https://i.imgur.com/58V5Dw7.png)

4. Check out the directory properties (SElinux Content) again!:
``` bash
ls -lZd /root/test
```
![](https://i.imgur.com/WCUoyMB.png)


> **REMEMBER!** If you mistype the fcontext type, it fails with *Invalid Argument* error! You should be careful!
> ![](https://i.imgur.com/zMcXuXS.png)

As you noticed, we have changed the fcontext of the directory from **admin_home_t** to **httpd_sys_content_t**. But remember, these changes are only temporary, after the reboot or using `restorecon`,it will be changed to admin_home_t fcontext type back again!



### Changing the fcontext of the file/dir **persistently!**

1. Add a second directory:
``` bash
mkdir /root/test2
```
2. Check out the second directory properties (SELinux content):
``` bash
# -l - long list
# -Z - SELinux content list
# -d - directory itself not the content of it
ls -lZd /root/test2
```
![](https://i.imgur.com/JSREou2.png)

3. Let's add 3 files (a,b,c) under the ./test2 directory:
``` bash
# {} - can be used to manipulate multiple files at once
touch /root/test2/{a,b,c}
```
4. Check out the second directory's content properties (SELinux content):
``` bash
# -l - long list
# -Z - SELinux content list
# -d - directory itself not the content of it
ls -lZ /root/test2/
```
![](https://i.imgur.com/0BhdCcm.png)

There is 3 files with the same fcontext type as their parent (./test2 dir). Sweet!

5. Now it's time to change the file context (fcontext) of our previously created directory /root/test2:
``` bash
#-a - add fcontext if it's not added yet. If it's added before, use "-m"
#-t - fcontext type
#"/root/test2(.*)?" - includes the files under ./test2 directory
semanage fcontext -a -t httpd_sys_content_t "/root/test2(/.*)?"
```

As you see, this command is different from the previous `chcon` that used to change the fcontext temporary. 

6. Check out the second directory properties (SElinux Content) again.
``` bash
ls -lZd /root/test2
```
![](https://i.imgur.com/BdlTqHo.png)

and the file content of ./test2 directory 
![](https://i.imgur.com/vWCT0Pu.png)


Oops! Nothing changed?! It's still **admin_home_t** under "type" column. That's because we should DO RESTORE MANUALLY after `semanage fcontext -a -t` command.

But before we restore the fcontext, let's check if there is any other fcontext that has been changed from default.

``` bash
semanage fcontext -Cl
```
![](https://i.imgur.com/pMY3lcu.png)

So you see, we have only one customized fcontext and it is previously changed the fcontext type of ./test2 directory. However, it's not restored (applied) yet. 

7. To restore (re-apply the fcontext) use:

``` bash
### Will ONLY change the fcontext type of the directory, NOT THE FILES/DIRS under it! 
#It's important
#-v - verbosity
restorecon -v /root/test2
```
![](https://i.imgur.com/oLiUVtw.png)


``` bash
### Will change the fcontext type of BOTH the directory and files under it.
#-v - verbosity
#-R - recursively (files/dirs under the current directory)
restorecon -vR /root/test2
```
![](https://i.imgur.com/QYYYZD8.png)

## Restore the file context (fcontext)

### Restoring the fcontext of the file/dir that was created **temporary!**

> **NOTE!** restore the fcontext type means to relabel a file/directory to its original SELinux context. Original state was defined by a parent directory or it's **default_t**!

1. Just reboot your computer or simply use `restorecon -v /path/to/dir/or/file`

``` bash
restorecon -v /root/test
```
![](https://i.imgur.com/68QqAKb.png)

So you see it's relabeled from **httpd_sys_content_t** to **admin_home_t** back! It's completely restored to the oroginal state!
### Restoring the fcontext of the file/dir that was created **persistently!**

1. To restore (return to the original state):
``` bash
#-d - restore the fcontext
semanage fcontext -d "/root/test2(/.*)?"
```
	As you see, nothing changed yet.
![](https://i.imgur.com/0knf6Rw.png)

2. After we restored the fcontext (returning to the original state), it's time to restore the fcontext.
``` bash
### Will ONLY change the fcontext type of the directory, NOT THE FILES/DIRS under it! 
#It's important
#-v - verbosity
restorecon -v /root/test2
```
![](https://i.imgur.com/2Mmiv3q.png)

To verify the changes, run `ls -lZd /root/test2` and `ls -lZ /root/test2` commands.
![](https://i.imgur.com/da7eQV8.png)

After we run both command, you have noticed that fcontext for directory is restored bu for files under it - DID NOT! So, to restore the context of **BOTH** the directory and files under it, use:

``` bash
### Will change the fcontext type of BOTH the directory and files under it.
#-v - verbosity
#-R - recursively (files/dirs under the current directory)
restorecon -vR /root/test2
```

![](https://i.imgur.com/wbnFpqn.png)

As you see now from the screenhot, both directories and files returned to the original fcontext SELinux state. (type: admin_home_t)

## Summary
___

## Change fcontext type:
1. **Temporary** - `chcon -t <fcontext type> -v [-R] /path/to/file/or/dir`
2. **Persistently** - `semanage fcontext -a [-m] -t <fcontext type> "/path/to/file/or/dir(/.*)?"`

## Restore to the default SELinux security contexts:
1. **Temporary** - *either* reboot the system *or* `restorecon -v /path/to/file/or/dir`
2. **Persistently** - `semanage fcontext -d "/path/to/file/or/dir(/.*)?"`

## Restore fcontext type after changes:
1. **Temporary** - no need, it's automatically restore its fcontext type untill the next boot or untill the usage of`restorecon` command.
2. **Persistently** - `restorecon -v [-R] /path/to/file/or/dir`

### P.S.

You can change the *fcontext* temporary or persistently, depends on the situation. If you don't want to lose the original (default) fcontext you will probably use temporary method:

**temporary method** -- To **change (rewrite)** the fcontext of the file or dir ***temporary***. (NOTE! It won't change the default fcontext of the file or dir found in /etc/selinux/targeted/contexts/files/file_contexts.local), use:

``` bash
chcon -t etc_t /anydir
```
To **return** to the original (default) fcontext of the file or dir found in /etc/selinux/targeted/contexts/files/file_contexts.local, use:

``` bash
restore /anydir
```

## Thank you!