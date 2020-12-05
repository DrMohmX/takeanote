---
layout: post
title: NFS və AutoFS
subtitle: Admin Mamedə NFS və AutoFS vasitasi ilə external serverdə /home directory yaradacaq...
tags: [rhel, exam, book, nfs, autofs]
gh-badge: [star, fork, follow]
comments: true
---
Salam, əziz oxucular. Bəri başdan demək istəyirəm ki, mən məktəbdə Azərbaycan fəni heç vaxt keçməmişəm, çunki Rusiyada məktəb oxumuşam, ona gorədə bəzi yerlərdə qramatik və orfoqrafik səhv etsəm, çox da şey eləməyin, bu Linux dərsliyidir, Azərbaycan dili yox. 

### Situasiya belədir.

İlk oncə iki nəfəri təqdim etmək istəyərdim:

10lar© şirkətin Sistem administratoru:

![](/assets/img/baner_002.png)

və 10lar© şirkətin Adi istifadəçisi:

![](/assets/img/baner_001.png)

Mamed başqa adi istifadəçilər kimi, kompu (device olaraq) tez-tez dəyişə bilər şirkət daxili. Ona görədə onun (və figər adi istifadiçilərin) butun fayılları (files) və qovluğları (directories) bir yerdə cəmləşməsi daha da məntiqlidir.Yani local olaraq kompda yox, məhz xarici serverdə. Yani xarici bir serverdə (external server) /home direktoriyası*Admin* yaradacaq və bütün gələcəkdə yaranan user-lər ("istifadəçi" əvəzi indən sonra "user" yazacam) serverdə yaranmış /home qovluğa "access"-i olacaq.

Bunun üçün biz **NFS, AutoFS, avtomatizasiya etmək üçün Simple bash** istifade edəcik. Admin və Mamed isə bizə bu test də köməklik edəcəklər.

