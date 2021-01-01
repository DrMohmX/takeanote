---
layout: post
title: Configure a network connection..
badge: /assets/img/basges/lab_asan.svg
subtitle: Task 2 (Tapşırıq 2).
tags: [rhel, exam, book, root]
gh-badge: [star, fork, follow]
comments: true

---
Salam, adminlər. İlk öncə deyib vurğulamaq istəyirəm ki, mən məktəbdə Azərbaycan fənni heç vaxt keçməmişəm, çunki Rusiyada məktəb oxumuşam, ona gorədə bəzi yerlərdə qramatik və orfoqrafik səhv etsəm, çox da şey eləməyin, bu Linux dərsliyidir, Azərbaycan dili yox. 

Share eləməyə unutmayın.


### İlk öncə

Hər bir "source" ilə istifadə edərək imtahana hazırlaşmalısınız.

### Context

Asghar Ghorinin əlbətdə bütün kitabını bura yerləşdirəsi deyiləm, bu müəlifin hüquqlarını tapdalıyır və kitabı ona görə də alib istifadə etmık lazımdır.

### Başlayaq

Ikinci task "Network connection" manually dəyişmək haqqındadır.

Bunu etmək üçün:

1. nmcli
2. nmtui


### 1. nmcli
nmcli tool'un bütün özəllikləri haqqında yazmıyacam, ətrafli bilmək üçün `man nmcli`'da öyrənə bilərsiz.

Sadəcə Network profile əlavə etmək üçün:

``` bash
sudo nmcli con add ifname enp0s3 con-name test_net type ethernet ipv4.address 192.168.0.70/24 ipv4.gateway 192.168.0.1 ipv4.dns 192.168.0.1,8.8.8.8
```
![](/assets/gifs/anime_4.gif)
p.s. sıralmanın **fərqi yoxdur**

* **con:** connection (full sözüdə yazmaq olar).
* **add:** şəbıkı profili əlavə etmək üçün (*up, down, delete, edit, reload, load* etc.).
* **ifname:** şəbəkə kartl kodirovkası (misal: enp0s3 - ethernet profile, wlp4s0 - wireless (WiFi) profile, virbr0 - virtual bridge, lo - localhost etc.). Məndə "enp0s3", ethernet profile yaratmaq üçün.
* **con-name:** şəbəkə adı. Məndə: test_net
* **type:** şəbəkə növə (ethernet, WiFi, VirtualBridge etc.)
* **ipv4.address:** IPv4 ünvanı (Class A - 10.XXX.XXX.XXX, Class B - 172.16-31.XXX.XXX, Class C - 192.168.XXX.XXX).
* **ipv4.gateway:** gateway (router) ünvanı.
* **ipv4.dns:** DNS server ünvanı(ları). **NOTE! Vergül istifadə edərək 2+ DNS ünvan əlavə etmək olar**

Yaradannan sonra, şəbəkəni **activeləşdirmək** üçün:
`nmcli con up test_net`

Deactivate üçün: 
`nmcli con down test_net`

### 2. nmtui

`nmtui` yazıb, Enter'a basırıq. That's all. Aşağda ki GIF'də eyni adlı şəbəkə yaradacam, sadəcə `nmtui` istifadə edərək.

![](/assets/gifs/anime_6.gif)

Mən iki metod haqqında yazdım,

#### Ancaq!

Gəlin diqqətlə oxuyaq: Using a manual method **(create/modify files by hand)**, configure a network connection on the primary network device with IP address 192.168.0.241/24, gateway 192.168.0.1, and nameserver 192.168.0.1.

Yanı əlimizdən `/etc/sysconfig/network-scripts/` papkas;na girib `ifcfg-test_net` file əlavə edib, parametirlər əlavə etməliyik. Heç bir halda, nə həyatda, nə imtahanda, nə intevyu da bunu etmək **LAZIM DEYİL**, yani ehtiyac yoxdur. Ancaq maraqlıdırsa, mənə yaza bilərsiz.

Bu qədər. Sağolun.