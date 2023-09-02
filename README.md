# Домашнее задание к занятию «Уязвимости и атаки на информационные системы»

### Задание 1

Скачайте и установите виртуальную машину Metasploitable: https://sourceforge.net/projects/metasploitable/.

Это типовая ОС для экспериментов в области информационной безопасности, с которой следует начать при анализе уязвимостей.

Просканируйте эту виртуальную машину, используя **nmap**.

Попробуйте найти уязвимости, которым подвержена эта виртуальная машина.

Сами уязвимости можно поискать на сайте https://www.exploit-db.com/.

Для этого нужно в поиске ввести название сетевой службы, обнаруженной на атакуемой машине, и выбрать подходящие по версии уязвимости.

Ответьте на следующие вопросы:

- Какие сетевые службы в ней разрешены?
- Какие уязвимости были вами обнаружены? (список со ссылками: достаточно трёх уязвимостей)
  
*Приведите ответ в свободной форме.*  

### Решение 1

Установил тестовую машину и nmap. Просканировал локальную сеть: 
```
nmap -sn 10.44.44.0/24
```
Нашел необходимую машину с адресом 10.44.44.7
Просканировал открытые порты машины:
```
root@ska-VirtualBox:/home/ska# nmap -sV 10.44.44.7
Starting Nmap 7.80 ( https://nmap.org ) at 2023-09-02 14:04 +05
Nmap scan report for 10.44.44.7
Host is up (0.000066s latency).
Not shown: 977 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login?
514/tcp  open  tcpwrapped
1099/tcp open  java-rmi    GNU Classpath grmiregistry
1524/tcp open  bindshell   Metasploitable root shell
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
MAC Address: 08:00:27:44:4C:9D (Oracle VirtualBox virtual NIC)
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
Из уязвимостей выделю:

5432/tcp open postgresql PostgreSQL DB 8.3.0 - 8.3.7
https://www.exploit-db.com/exploits/32849

21/tcp open ftp vsftpd 2.3.4
https://www.exploit-db.com/exploits/49757
https://www.exploit-db.com/exploits/17491

53/ПРИВЯЗКА ISC к открытому домену tcp 9.4.2
https://www.exploit-db.com/exploits/6122

### Задание 2

Проведите сканирование Metasploitable в режимах SYN, FIN, Xmas, UDP.

Запишите сеансы сканирования в Wireshark.

Ответьте на следующие вопросы:

- Чем отличаются эти режимы сканирования с точки зрения сетевого трафика?
- Как отвечает сервер?

*Приведите ответ в свободной форме.*

### Решение 2

SYN метод отличается тем что он может быть быстро запущен, он способен сканировать тысячи портов в секунду при быстром соединении, его работе не препятствуют ограничивающие бранмауэры. Этот тип сканирования относительно ненавящив и незаметен, т.к. при таком сканировании TCP соединение никогда не устанавливается до конца. Он работает с любым TCP стеком, не завися от каки-либо особенностей специфичной платформы, как это происходит при сканированиях типа FIN/NULL/Xmas, Maimon и idle сканировании. 

Запускаем командой:
```
nmap -sS 10.44.44.7
```
В Wireshark видим:
![1](https://github.com/SKA1010/sec_1/assets/125235217/7a007a2b-7244-45ea-a7b6-d5e200214bf7)

В FIN методе за основу взяты некоторые тонкости реализации протокола TCP в различных сетевых ОС: на передаваемый TCP FIN-запрос закрытые порты отвечают пакетом с флагом RST, а открытые порты данное сообщение игнорируют. Однако сетевые ОС фирмы Microsoft таким методом просканировать не удастся, так как в их реализации протокола TCP передача пакета TCP RST в ответ на подобный запрос не предусмотрена.

Запускаем командой:
```
nmap -sF 10.44.44.7
```
В Wireshark видим:
![2](https://github.com/SKA1010/sec_1/assets/125235217/6bd3cdcc-32bd-4366-9c9b-de2233de6450)

Xmas Scan — сканирование, которое основано на отправке пакетов, содержащих на каждый запрос разные флаги из всех доступных для контроля соединения.

Запускаем командой:
```
nmap -sX 10.44.44.7
```
В Wireshark видим:
![3](https://github.com/SKA1010/sec_1/assets/125235217/562b7535-e03c-4fe8-a5ee-ad9026a71e11)

UDP-сканирование работает путем отправки UDP-пакета на каждый целевой порт. Для большинства портов этот пакет будет пустым (без полезной нагрузки), но для нескольких наиболее распространенных портов будет отправлена полезная нагрузка, зависящая от протокола.

Запускаем командой:
```
nmap -sU 10.44.44.7
```
В Wireshark видим:
![4](https://github.com/SKA1010/sec_1/assets/125235217/f3009d09-3601-4617-b35b-7a15d02c3880)

