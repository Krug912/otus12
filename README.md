# Инициализация системы. Systemd
1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
2. Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).
3. Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.
### 1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
```
root@system:~# history
    1  vi /etc/default/watchlog
    2  cat /etc/default/watchlog
    3  vi /var/log/watchlog.log
    4  vi /opt/watchlog.sh
    5  chmod +x /opt/watchlog.sh
    6  vi /etc/systemd/system/watchlog.service
    7  vi /etc/systemd/system/watchlog.timer
    8  systemctl start watchlog.timer
    9  tail -n 100 /var/log/syslog
   10  history

root@system:~$ systemctl list-timers --all
NEXT                        LEFT        LAST                        PASSED               UNIT                  >
Wed 2025-01-15 17:17:31 UTC 22s left    Wed 2025-01-15 17:17:01 UTC 7s ago               watchlog.timer        >
Thu 2025-01-16 00:00:00 UTC 6h left     n/a                         n/a                  dpkg-db-backup.timer  >
Thu 2025-01-16 00:00:00 UTC 6h left     Wed 2025-01-15 16:50:20 UTC 26min ago            logrotate.timer       >
Thu 2025-01-16 00:24:57 UTC 7h left     Wed 2025-01-15 17:01:40 UTC 15min ago            fwupd-refresh.timer   >
Thu 2025-01-16 04:29:44 UTC 11h left    Tue 2024-07-23 18:02:42 UTC 5 months 23 days ago man-db.timer          >
Thu 2025-01-16 10:38:16 UTC 17h left    Wed 2025-01-15 17:05:21 UTC 11min ago            motd-news.timer       >
Thu 2025-01-16 17:05:21 UTC 23h left    Wed 2025-01-15 17:05:21 UTC 11min ago            systemd-tmpfiles-clean>
Sun 2025-01-19 03:10:34 UTC 3 days left Wed 2025-01-15 16:50:39 UTC 26min ago            e2scrub_all.timer     >
Mon 2025-01-20 00:48:24 UTC 4 days left Wed 2025-01-15 17:00:45 UTC 16min ago            fstrim.timer          >
n/a                         n/a         n/a                         n/a                  apport-autoreport.time>
n/a                         n/a         n/a                         n/a                  snapd.snap-repair.time>
n/a                         n/a         n/a                         n/a                  ua-timer.timer        >

12 timers listed.
lines 1-15/15 (END)

root@system:~# tail -n 1000 /var/log/syslog  | grep word
Jan 15 16:50:21 vagrant kernel: [   16.069458] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
Jan 15 17:11:33 vagrant root: Wed Jan 15 05:11:33 PM UTC 2025: I found word, Master!
Jan 15 17:12:21 vagrant root: Wed Jan 15 05:12:21 PM UTC 2025: I found word, Master!
Jan 15 17:12:54 vagrant root: Wed Jan 15 05:12:54 PM UTC 2025: I found word, Master!
Jan 15 17:13:59 vagrant root: Wed Jan 15 05:13:59 PM UTC 2025: I found word, Master!
Jan 15 17:15:21 vagrant root: Wed Jan 15 05:15:21 PM UTC 2025: I found word, Master!
Jan 15 17:16:11 vagrant root: Wed Jan 15 05:16:11 PM UTC 2025: I found word, Master!
Jan 15 17:17:01 vagrant root: Wed Jan 15 05:17:01 PM UTC 2025: I found word, Master!
Jan 15 17:17:36 vagrant root: Wed Jan 15 05:17:36 PM UTC 2025: I found word, Master!

```
### 2. Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).
```
   20  apt install spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid -y
   21  vi spawn-fcgi.sh
   22  vi /etc/spawn-fcgi/fcgi.conf
   23  cat /etc/spawn-fcgi/fcgi.conf
   24  mkdir /etc/spawn-fcgi/
   25  vi /etc/spawn-fcgi/fcgi.conf
   26  vi /etc/systemd/system/spawn-fcgi.service
   27  systemctl start spawn-fcgi
   28  systemctl status spawn-fcgi
   29  history
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
     Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2025-01-15 17:26:09 UTC; 7s ago
   Main PID: 11674 (php-cgi)
      Tasks: 33 (limit: 2218)
     Memory: 14.1M
        CPU: 68ms
     CGroup: /system.slice/spawn-fcgi.service
             ├─11674 /usr/bin/php-cgi
             ├─11676 /usr/bin/php-cgi
             ├─11677 /usr/bin/php-cgi
             ├─11678 /usr/bin/php-cgi
             ├─11679 /usr/bin/php-cgi
             ├─11680 /usr/bin/php-cgi
             ├─11681 /usr/bin/php-cgi
             ├─11682 /usr/bin/php-cgi
             ├─11683 /usr/bin/php-cgi
             ├─11685 /usr/bin/php-cgi
             ├─11686 /usr/bin/php-cgi
             ├─11687 /usr/bin/php-cgi
             ├─11688 /usr/bin/php-cgi
             ├─11689 /usr/bin/php-cgi
             ├─11690 /usr/bin/php-cgi
             ├─11691 /usr/bin/php-cgi
             ├─11692 /usr/bin/php-cgi
             ├─11693 /usr/bin/php-cgi
             ├─11694 /usr/bin/php-cgi
             ├─11695 /usr/bin/php-cgi
             ├─11696 /usr/bin/php-cgi
             ├─11697 /usr/bin/php-cgi
             ├─11698 /usr/bin/php-cgi
             ├─11699 /usr/bin/php-cgi
             ├─11700 /usr/bin/php-cgi
             ├─11701 /usr/bin/php-cgi
             ├─11702 /usr/bin/php-cgi
             ├─11703 /usr/bin/php-cgi
             ├─11704 /usr/bin/php-cgi
             ├─11705 /usr/bin/php-cgi
             ├─11706 /usr/bin/php-cgi
             ├─11707 /usr/bin/php-cgi
             └─11708 /usr/bin/php-cgi

Jan 15 17:26:09 system systemd[1]: Started Spawn-fcgi startup service by Otus.
```
### 3. Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.
```
   30  apt install nginx -y
   31  vi /etc/systemd/system/nginx@.service
   32  vi /etc/nginx/nginx-first.conf
   33  vi /etc/systemd/system/nginx@.service
   34  cat /etc/nginx/nginx.conf
   35  vi /etc/nginx/nginx-first.conf
   36  vi /etc/nginx/nginx-second.conf
   37  cat /etc/nginx/nginx-first.conf
   38  vi /etc/nginx/nginx-second.conf
   39  systemctl start nginx@first
   40  journalctl -xeu nginx@first.service
   41  vi /etc/nginx/nginx-second.conf
   42  cat /etc/nginx/nginx.conf
   43  vi /etc/nginx/nginx-second.conf
   44  vi /etc/nginx/nginx-first.conf
   45  systemctl start nginx@first
   46  systemctl start nginx@second
   47  systemctl status nginx@first
   48  systemctl status nginx@system
   49  systemctl status nginx@second
   50  systemctl status nginx@first
   51  history
root@system:~# systemctl status nginx@system
○ nginx@system.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:nginx(8)
root@system:~# systemctl status nginx@second
● nginx@second.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2025-01-15 17:45:13 UTC; 34s ago
       Docs: man:nginx(8)
    Process: 12449 ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-second.conf -q -g daemon on; master_proc>
    Process: 12450 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on; (>
   Main PID: 12451 (nginx)
      Tasks: 3 (limit: 2218)
     Memory: 3.3M
        CPU: 61ms
     CGroup: /system.slice/system-nginx.slice/nginx@second.service
             ├─12451 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; maste>
             ├─12452 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >
             └─12453 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >

Jan 15 17:45:13 system systemd[1]: Starting A high performance web server and a reverse proxy server...
Jan 15 17:45:13 system systemd[1]: Started A high performance web server and a reverse proxy server.
root@system:~# systemctl status nginx@first
● nginx@first.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2025-01-15 17:45:05 UTC; 49s ago
       Docs: man:nginx(8)
    Process: 12440 ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-first.conf -q -g daemon on; master_proce>
    Process: 12443 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master_process on; (c>
   Main PID: 12444 (nginx)
      Tasks: 3 (limit: 2218)
     Memory: 3.4M
        CPU: 58ms
     CGroup: /system.slice/system-nginx.slice/nginx@first.service
             ├─12444 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master>
             ├─12445 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >
             └─12446 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >

Jan 15 17:45:05 system systemd[1]: Starting A high performance web server and a reverse proxy server...
Jan 15 17:45:05 system systemd[1]: Started A high performance web server and a reverse proxy server.

root@system:~# ss -tulpan
Netid      State        Recv-Q       Send-Q              Local Address:Port             Peer Address:Port       Process
udp        UNCONN       0            0                   127.0.0.53%lo:53                    0.0.0.0:*           users:(("systemd-resolve",pid=649,fd=13))
udp        UNCONN       0            0                  10.0.2.15%eth0:68                    0.0.0.0:*           users:(("systemd-network",pid=1300,fd=18))
tcp        LISTEN       0            511                       0.0.0.0:8080                  0.0.0.0:*           users:(("nginx",pid=12446,fd=6),("nginx",pid=12445,fd=6),("nginx",pid=12444,fd=6))
tcp        LISTEN       0            511                       0.0.0.0:8081                  0.0.0.0:*           users:(("nginx",pid=12453,fd=6),("nginx",pid=12452,fd=6),("nginx",pid=12451,fd=6))
tcp        LISTEN       0            128                       0.0.0.0:22                    0.0.0.0:*           users:(("sshd",pid=933,fd=3))
tcp        LISTEN       0            4096                127.0.0.53%lo:53                    0.0.0.0:*           users:(("systemd-resolve",pid=649,fd=14))
tcp        ESTAB        0            0                       10.0.2.15:22                   10.0.2.2:53258       users:(("sshd",pid=1672,fd=4),("sshd",pid=1633,fd=4))
tcp        LISTEN       0            128                          [::]:22                       [::]:*           users:(("sshd",pid=933,fd=4))
tcp        LISTEN       0            511                             *:80                          *:*           users:(("apache2",pid=11423,fd=4),("apache2",pid=11422,fd=4),("apache2",pid=11421,fd=4),("apache2",pid=11420,fd=4),("apache2",pid=11419,fd=4),("apache2",pid=11417,fd=4),("apache2",pid=11416,fd=4))

```
