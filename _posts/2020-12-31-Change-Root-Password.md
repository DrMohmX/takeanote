---
layout: post
title: Root Password dəyişdirilmısi...
subtitle: Task 1 (Tapşırıq 1).
tags: [rhel, exam, book, root]
gh-badge: [star, fork, follow]
comments: true

---
Salam, adminlər. İlk öncə deyib vurğulamaq istəyirəm ki, mən məktəbdə Azərbaycan fənni heç vaxt keçməmişəm, çunki Rusiyada məktəb oxumuşam, ona gorədə bəzi yerlərdə qramatik və orfoqrafik səhv etsəm, çox da şey eləməyin, bu Linux dərsliyidir, Azərbaycan dili yox. 

Share eləməyə unutmayın.


![](/assets/gifs/anime_3.gif)


### İlk öncə

Hər bir "source" ilə istifadə edərək imtahana hazırlaşmalısınız.

### Context

Asghar Ghorinin əlbətdə bütün kitabını bura yerləşdirəsi deyiləm, bu müəlifin hüquqlarını tapdalıyır və kitabı ona görə də alib istifadə etmık lazımdır.

### Başlayaq

Birinci task "unudulmuş" root parolunu dəyişdirilməsi ilə bağlıdır. Bunun üçün yeganə və asan yol var.

Hostu restart edirsiniz və mövcud (adətən sonuncu olan) kernel-u seçib `e` düyməsinə basırsınız. (məndə "4.18.0-240.8.1.el8_3" versiyalı kernel-dır)

![](/assets/img/screenshots/Screen_0014.png)

Daxil olduğunüz Kernel settings-dir və içində olanlar - **Kernel parameters**-lərdi. Burada fərqli kernel-in **boot** olunmasını edit etmək olar. Və hər hansı bir kernel (məndə 2 dənədir) halsız vəziyyətə düşsə, digərini start etmək olar. Məhz ona görə müxtəlif versiyali Kernellar var bootloaderdə. Heç bir kernel işləmirsə, artıq Live cd istifadə edərək həl eləmək olar, odda artıq başqa mövzudur. 

Parolu dəyişmək üçün gərək kernel prosesini dayandırmaq (break) lazımdır. Break eləməsək,`/sysroot` qovluğu (*indidən sonra "dir" yazacam*) mount olunacaq. Biz `rd.break` yazmaqla, elə bilki kernel tərfindən təbii olan mount prossesini **dayandırıq!** 

p.s. linux boot addımları

![](/assets/img/img_1.jpg)

Gördüyümüz kimi şəkildə --> [ BIOS - MBR - GRUB - Ker -- **X (break noqtəsi)** --nel - Init - Runlevel ] və `rd.break` istifadə edərkən, Kernel mount edib, "İnit" addımına **getmir**.


* `linux` ilə başlayan line-nın sonuna, **(1.) `rd.break`** əlavə edirik və **(2.) `Ctrl-x` basırıq**. 

![](/assets/gifs/anime_1.gif)

p.s. `linux` ilə başlayan line-da olan digər parameters-lər haqqında, [burada](https://takeanote.info) izah etmişəm.
<br>
* System load olunub, `switch_root`'a daxil olur. 

    **(3.) `mount -o remount,rw /sysroot`**
*-o* - options
*remount* -- **umount** edib, sonra **mount** edir. 
*rw* --------- read/write

    Burada biz, `/sysroot` dir'i (bu dir'də bütün system filelarımız yerləşir) **remount** edirik. `/sysroot`'u `/` (root) kimi qəbul edin, `switch_root` `/` (root'u), `/sysroot` kimi görür. Vəssalam.
<br>
* Novbəti addımda, ** (4.)`chroot /sysroot`** komandası istifadə edərkən, root fs-i `/sysroot`'a dəyişirik. Buda o deməkdir ki, artıq biz systemdəyik, və bütün kompun filelarına access var.
<br>
* Bu mərhələdə biz artıq parolu dəyişə bilərik.

    **(5.) `passwd`** və "1234" (yeni parol; fərqli ola bilər) yazırıq.

    *or*

    **echo "1234" | passwd --stdin root**

* Axırda isə, **(6.) `touch /.autorelabel`** komandası istifadə edib, root-da `.autorelabel` file yaradırıq ki, SELinux komandası kompda olan nodesların labelarını dəyişsin (SELinux - başqa mövzudur).
<br>
* **(7.) `Ctrl-D` `Ctrl-D`**
<br>
![](/assets/gifs/anime_2.gif)
* **
### Summary

1. `e` --> linux ..... + **rd.break**
2. `Ctrl-x`
3. `mount -o remount,rw /sysroot`
4. `chroot /sysroot`
5. `echo "1234" | passwd --stdin root`
6. `touch /.autorelabel`
7. 2x `Ctrl-d` 
<br>
