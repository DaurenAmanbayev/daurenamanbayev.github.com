---
layout: post
title: Glassfish 4.1.1 start-domain failed
categories: Troubleshooting
---

При инсталляции на Windows 10 сервера Glassfish у меня возникла проблема с запуском домена по умолчанию (asadmin start-domain domain1): start-domain failed. 

Для того, чтобы запустить таки сервер, требуется создать собственный домен  с именем отличным от domain1:
* asadmin create-domain mydomain
* asadmin start-domain mydomain














