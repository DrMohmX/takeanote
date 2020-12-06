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


### 1 - NFS Utils install edilməsi və aktvləşdirilməsi...

Sonra **NFS**-i install edir...

``` bash
sudo yum install -y nfs-utils
```
![](/assets/img/screenshots/Screen_0001.png)

	`sudo systemctl enable --now nfs-server.service`

![](/assets/img/screenshots/Screen_0010.png)

### 3 - /home Qovluğu export (share; bölüşməsi) edilməsi...

a)
``` bash
sudo vim /etc/exports
```
	/home		192.168.0.121(rw,no_root_squash)

line-i əlavə edir.
![](/assets/img/screenshots/Screen_0007.png)

*yada*

b)
``` bash
#bir yazı ilə exports file-ına əlavə etdi
sudo echo "/home	192.168.0.121(rw,no_root_squash)" >> /etc/exports
```
Admin "/home" qovluğü 192.168.0.121 ipv4-li client (kompyuter) ilə bölüşdü, client (kompyuter) oxuyub/yaza (read/write) bilər "/home"-da, və clientin "root" istifadəçisi serverin "root" istifadəçisi kimi hər şey edə bilər bu qovluğda.

Bu line-i elavə edəndən sonra,

1. Clientlətin (xaricdə olanların) bu NFS qovluğuna (/home) qoşulmaq üçün, gərək Admin Firewall-a 3 dənə sevice əlavə etsin:

``` bash
sudo firewall-cmd --add-service nfs --permanent
sudo firewall-cmd --add-service mountd --permanent
sudo firewall-cmd --add-service rpc-bind --permanent
sudo firewall-cmd --reload
```


2. `sudo exportfs -av`
![](/assets/img/screenshots/Screen_0008.png)

Bu bolüşmə (export, share) işini tamamlayır.

p.s. 

ipv4 192.168.0.121 əvəzinə əsas adla (alias ilə) istifadə edirlər, onun üçün:
``` bash
sudo echo "192.168.0.121 	client1" >> /etc/hosts
```

Serverdə bu qədər.

## Clientdə

### 1 - NFS Utils install edilməsi...

**NFS**-i install edir...

``` bash
sudo yum install -y nfs-utils
```
![](/assets/img/screenshots/Screen_0002.png)

Admin install edəndən sonra, yoxlamaq istəyir ki Serverdə olan "/home" qovluğ Clientdə görünür ya yox, bunun üçün:
`showmount -e 192.168.0.120`
![](/assets/img/screenshots/Screen_0011.png)

Əla! Serverdəki qovluğ gorünür.
### 2 - AutoFS install edilməsi və aktivləşdirilməsi...

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

### 3 - /etc/auto.master edit edilməsi...

Ən vacib addımlardan biri, burda çox diqqətli olmalıdır Admin. O bu addımda Client komyuterində qovluğ yaradıb, və yeni istifadiçəlir o qovluğla "home base" kimi istifadə etməlidirlər. Yaradılmış qovluğ **statik(!)** olacaq, onun altında avtomatik yaradılmış qovluğlar isə **dynamic(!)**. Demeli, ilk öncə /etc/auto.master file-ı edit edir admin:

``` bash
sudo vim /etc/auto.master
```
![](/assets/img/screenshots/Screen_0005.png)

Orda "/misc   /etc/auto.misc" mövcuddur, ona əl vurmur, onun altinda isə yeni line əlavə edir:

	/clients 	/etc/auto.master.d/auto.clients


![](/assets/img/screenshots/Screen_0006.png)

Bu line onu deyir ki, **statik** olaraq "/clients" qovluğu client kompyuterində yaradılır, və /etc/auto.master.d/auto.clients faylındaki settingslərə uyuğun olaraq, "/clients" qovluğun altında **dynamic** işlər görülür (yani, dynamic olaraq sub-qovluğlar yaranır).

p.s. 
1. /etc/auto.master.d altında yaradılması daha məsləhtlidir, soruşma niyə, fərqli mövzudur.
2. auto.clients - fərqli adda ola bilər, məsələn "auto.klient", "auto.client", "auto.mushteri". Onun fərqi yoxdur.

### 4 - /etc/auto.master.d/auto.clients yaradılması və edit edilməsi...

Bu addımda vacibdir, çünki bu addımda Admin elə bir line əlavə etməlidir ki "/etc/auto.master.d/auto.clients" faylına, adınnan fərqli olmuyaraq, hər bir userin "/clients" qovluğun altında ONA AYID HOME DİR olsun. Məsələn, "mamed" adlı userin home diri olsun /clients/mamed; "kamala" --- /clients/kamala ; "arzu" --- /clients/arzu və s.

``` bash
sudo vim /etc/auto.master.d/auto.clients
```

	*		-rw		192.168.0.120:/home/&
	
<br>asterisk(\*) - bütün userlər.
<br>-rw - read/write.
<br>192.168.0.120 - Server IP.
<br>:/home - Serverdəki export (share) olunmuş "/home"qovluğu.
<br>& - Userin adına uyğun olaraq, sub-qovluğ yaranacaq.

![](/assets/img/screenshots/Screen_0012.png)

Bu line əlavə edəndən sonra, AutoFS restart etmək lazımdır.

`sudo systemctl restart autofs.service`

Restart etdikdən sonra, görmək olar ki "/clients" qovluğu yarandı (`mkdir` siz, avtomatik olaraq, /etc/auto.master -də "/clients ......" yazdığımız üçün)

`ls -l /`

![](/assets/img/screenshots/Screen_0013.png)

Bu qədər. İndi qaldı Serverdə və Clientdə user yaradıb yoxlamaq.

Onun üçün Admin simple bash yazır.


Script belədir:

``` bash
#!/bin/bash

read -p "Useri yaz: " user
read -p "UIDini yaz, manual olaraq: " uid
read -s -p "Parolu yaz $user uchun: " pass

if [[ -z $uid ]]; then

	useradd $user
	echo "$pass" | passwd --stdin $user
	ssh root@192.168.0.121 useradd -Mb /clients $user 
	ssh root@192.168.0.121 echo "$pass" | passwd --stdin $user
else
	useradd -u $uid $user
	echo "$pass" | passwd --stdin $user
	ssh root@192.168.0.121 useradd -u $uid -Mb /clients $user 
	ssh root@192.168.0.121 echo "$pass" | passwd --stdin $user
fi
```

Suallarınız varsa: <br>
mail: <kerimov.rustam@live.ru> <br>
linkedin: [burada yaza bilərsən](https://www.linkedin.com/in/rustam-karimov-7293315b/)