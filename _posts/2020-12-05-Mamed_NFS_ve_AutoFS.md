---
layout: post
title: NFS və AutoFS
subtitle: Admin Mamedə NFS və AutoFS vasitasi ilə external serverdə /home directory yaradacaq...
tags: [rhel, exam, book, nfs, autofs]
gh-badge: [star, fork, follow]
comments: true
---
Salam, əziz oxucular. İlk öncə deyib vurğulamaq istəyirəm ki, mən məktəbdə Azərbaycan fənni heç vaxt keçməmişəm, çunki Rusiyada məktəb oxumuşam, ona gorədə bəzi yerlərdə qramatik və orfoqrafik səhv etsəm, çox da şey eləməyin, bu Linux dərsliyidir, Azərbaycan dili yox. 

### Situasiya belədir.

İlk oncə iki nəfəri təqdim etmək istəyərdim:

10lar© şirkətin Sistem administratoru:

![](/assets/img/baner_002.png)

və 10lar© şirkətin Adi istifadəçisi:

![](/assets/img/baner_001.png)

Mamed başqa adi istifadəçilər kimi, kompu (device olaraq) tez-tez dəyişə bilər şirkət daxili. Ona görədə onun (və figər adi istifadiçilərin) butun fayılları (files) və qovluğları (directories) bir yerdə cəmləşməsi daha da məntiqlidir.Yani local olaraq kompda yox, məhz xarici serverdə. Yani xarici bir serverdə (external server) /home direktoriyası *Admin* yaradacaq və bütün gələcəkdə yaranan user-lər ("istifadəçi" əvəzi indən sonra "user" yazacam) serverdə yaranmış /home qovluğa "access"-i olacaq.

Bunun üçün biz **NFS, AutoFS və avtomatizasiya etmək üçün Simple bash** istifade edəcəyik. Admin və Mamed isə bizə bu test də köməklik edəcəklər.

İki (2) dənə VM təqdim edirəm. Onun biri **Admin VM**, digəri isə **User VM** (bizim halda, Mamedin). 

1 VM (Server - multi-user): Linux, ***CentOS***, RAM: 4Gb, 2x2 CPU (Ryzen 7 2700x), kernel-version: 4.18.0-193, 
Network settings: 
**ipv4 192.168.0.120/24** 
gateway 192.168.0.100
dns 192.168.0.100

2 VM (Client - graphic): Linux, ***RHEL 8***, RAM: 4Gb, 2x2 CPU (Ryzen 7 2700x), kernel version: 4.18.0-193,
Network settings: 
**ipv4 192.168.0.121/24**
gateway 192.168.0.100
dns 192.168.0.100


### Başlayaq...

Axşam saat 11 dir, *Admin* qalıb işdə, IT Menecer ona tapşırıb ki, sabahdan şirkətdə yaradılmış bütün userlərin external serverdə (VM 1) eyni BİR qovluğü olmalıdır, o qovluğun altında (umbrella), hər usera ayıd (adları ilə eyni) home directory yaranmalıdır. Admin bir az fikirləşib, "Spotify"-da "The Godfather" soundtrackləri qoşub, başlayıb işə..

## Serverdə

### 1 - Ümümi qovluğun yaradılması...

Admin ilk oncə serverdə ümumi directory yaradır:

``` bash
sudo mkdir /homes
```

adı "/homes" qoyur çunki "home" artıq var. Onun belə fərqi yoxdur.

### 2 - NFS Utils install edilməsi...

Sonra **NFS**-i install edir...

``` bash
sudo yum install -y nfs-utils
```
![](/assets/img/screenshots/Screen_0001.png)

### 3 - Qovluğu export (share; bölüşməsi) edilməsi...

a)
``` bash
sudo vim /etc/exports
```
	/homes		192.168.0.121(rw,no_root_squash)

line-i əlavə edir.

*yada*

b)
``` bash
#bir yazı ilə exports file-ına əlavə etdi
sudo echo "/homes	192.168.0.121(rw,no_root_squash)" >> /etc/exports
```
Admin "/homes" qovluğü 192.168.0.121 ipv4-li client (kompyuter) ilə bölüşdü, client (kompyuter) oxuyub/yaza (read/write) bilər "/homes"-da, və clientin "root" istifadəçisi serverin "root" istifadəçisi kimi hər şey edə bilər bu qovluğda.

P.s. 

ipv4 192.168.0.121 əvəzinə əsas ad (alias) ilə istifadə edirlər, onun üçün:
``` bash
echo "192.168.0.121 	client1" >> /etc/hosts
```

Serverdə bu qədər.

## Clientdə

### 1 NFS Utils install edilməsi...

**NFS**-i install edir...

``` bash
sudo yum install -y nfs-utils
```
![](/assets/img/screenshots/Screen_0002.png)

### 2 AutoFS install edilməsi və aktivləşdirilməsi...

Admin ANCAQ client kompyuterində AutoFS tool-u install edir, ancaq clientdə (YADDA SAXLA)

``` bash
sudo yum install -y autofs 
```
![](/assets/img/screenshots/Screen_0003.png)

AutoFS install edib qurtarandan sonra, Admin onu mütləq aktiv (enable, start; enable --now) etməlidir. Onun üçün:

``` bash
sudo systemctl enable --now autofs.service 
```
![](/assets/img/screenshots/Screen_0004.png)

### 3 /etc/auto.master edit edilməsi...

Ən vacib addımlardan biri, burda çox diqqətli olmalıdır Admin. O bu addımda Client komyuterində qovluğ yaradıb, və yeni istifadiçəlir o qovluğla "home base" kimi istifadə etməlidirlər. Yaradılmış qovluğ **statik(!)** olacaq, onun altında avtomatik yaradılmış qovluğlar isə **dynamic(!)**. Demeli, ilk öncə /etc/auto.master file-ı edit edir admin:

``` bash
sudo vim /etc/auto.master
```
![](/assets/img/screenshots/Screen_0005.png)

Orda "/misc   /etc/auto.misc" mövcuddur, ona əl vurmur, onun altinda isə yeni line əlavə edir:

	/clients 	/etc/auto.master.d/auto.clients


![](/assets/img/screenshots/Screen_0006.png)

Bu line onu deyir ki, **statik** olaraq "/clients" qovluğu client kompyuterində yaradılır, və /etc/auto.master.d/auto.clients faylındaki settingslərə uyuğun olaraq, "/clients" də **dynamic** işlər görülür (yani, sub-qovluğlar yaranır)

p.s 
1. /etc/auto.master.d altında yaradılması daha məsləhtlidir, soruşma niyə, fərqli mövzudur
2. auto.clients - fərqli adda ola bilər, məsələn "auto.klient", "auto.client", "auto.mushteri". Onun fərqi yoxdur.

### 4 /etc/auto.master.d/auto.clients yaradılması və edit edilməsi...

