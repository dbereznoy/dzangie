## Подключение официального репозитория Angie и установка Angie Pro + 2 модуля
## Установите Angie и несколько дополнительных (на выбор) модулей из репозитория
root@angie-test:~# cd /home/berezhnoidv/

root@angie-test:/home/berezhnoidv# cp angie* /etc/ssl/angie

root@angie-test:/home/berezhnoidv# sudo curl -o /etc/apt/trusted.gpg.d/angie-signing.gpg \
            https://angie.software/keys/angie-signing.gpg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4491  100  4491    0     0  69623      0 --:--:-- --:--:-- --:--:-- 70171

root@angie-test:/home/berezhnoidv# echo "deb https://download.angie.software/angie-pro/$(. /etc/os-release && echo "$ID/$VERSION_ID $VERSION_CODENAME") main" \
    | sudo tee /etc/apt/sources.list.d/angie.list > /dev/null
    
root@angie-test:/home/berezhnoidv# cat /etc/apt/apt.conf.d/90download-angie
Acquire::https::download.angie.software::Verify-Peer "true";
Acquire::https::download.angie.software::Verify-Host "true";
Acquire::https::download.angie.software::SslCert     "/etc/ssl/angie/angie-repo.crt";
Acquire::https::download.angie.software::SslKey      "/etc/ssl/angie/angie-repo.key";

root@angie-test:/home/berezhnoidv# apt update

root@angie-test:/home/berezhnoidv# apt install angie-pro angie-pro-module-brotli angie-pro-module-perl

root@angie-test:/home/berezhnoidv# systemctl status angie
● angie.service - Angie - high performance web server
     Loaded: loaded (/usr/lib/systemd/system/angie.service; enabled; preset: enabled)
     Active: active (running) since Thu 2025-05-22 11:57:33 UTC; 16min ago
       Docs: https://en.angie.software/angie/docs/
    Process: 786 ExecStart=/usr/sbin/angie -c /etc/angie/angie.conf (code=exited, status=0/SUCCESS)
   Main PID: 802 (angie)
      Tasks: 2 (limit: 2316)
     Memory: 3.8M (peak: 4.3M)
        CPU: 13ms
     CGroup: /system.slice/angie.service
             ├─802 "angie: master process v1.9.0 PRO #1 [/usr/sbin/angie -c /etc/angie/angie.conf]"
             └─804 "angie: worker process #1"

May 22 11:57:33 angie-test systemd[1]: Starting angie.service - Angie - high performance web server...
May 22 11:57:33 angie-test systemd[1]: Started angie.service - Angie - high performance web server.

## Запустите Angie из Docker-образа. Необходимо вынести конфигурацию в хостовую директорию и указать проброс портов в хостовую сеть для HTTP/HTTPS
root@angie-test:/home/berezhnoidv# apt install docker.io

root@angie-test:/home/berezhnoidv# docker run --rm --name angie -v /var/www:/usr/share/angie/html:ro  -p 8080:80 -d docker.angie.software/angie:latestc6fa1a3545564d6036ab9020f24a3a52582d814abe4c7d57e367962c2de0f48c

root@angie-test:/home/berezhnoidv# docker ps
CONTAINER ID   IMAGE                                COMMAND                  CREATED         STATUS         PORTS                                   NAMES
c6fa1a354556   docker.angie.software/angie:latest   "angie -g 'daemon of…"   5 seconds ago   Up 4 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp   angie

root@angie-test:/home/berezhnoidv# netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:41041         0.0.0.0:*               LISTEN      2123/containerd
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      802/angie: master p
tcp        0      0 127.0.0.1:6010          0.0.0.0:*               LISTEN      1238/sshd: berezhno
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      2725/docker-proxy
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      470/systemd-resolve
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      470/systemd-resolve
tcp6       0      0 ::1:6010                :::*                    LISTEN      1238/sshd: berezhno
tcp6       0      0 :::22                   :::*                    LISTEN      1/init
tcp6       0      0 :::8080                 :::*                    LISTEN      2731/docker-proxy
udp        0      0 127.0.0.54:53           0.0.0.0:*                           470/systemd-resolve
udp        0      0 127.0.0.53:53           0.0.0.0:*                           470/systemd-resolve
udp        0      0 10.129.0.32:68          0.0.0.0:*                           752/systemd-network

root@angie-test:/home/berezhnoidv# cat /var/www/index.html
<h2>test docker</h2>

root@angie-test:/home/berezhnoidv# curl localhost:8080
<h2>test docker</h2>

root@angie-test:/home/berezhnoidv# docker rm -f angie
angie

root@angie-test:/home/berezhnoidv# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@angie-test:/home/berezhnoidv# docker run  --name angie -v /var/www:/usr/share/angie/html:ro -v /home/berezhnoidv/angie:/etc/angie:ro -p 8080:80 -d docker.angie.software/angie:latest
a4cd3875bdfd4e4be2093214e4a7b1956824dcbace0bd994d353aa3e94cfb7fb

root@angie-test:/home/berezhnoidv# docker ps
CONTAINER ID   IMAGE                                COMMAND                  CREATED         STATUS         PORTS                                   NAMES
a4cd3875bdfd   docker.angie.software/angie:latest   "angie -g 'daemon of…"   9 seconds ago   Up 9 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp   angie
root@angie-test:/home/berezhnoidv# curl localhost:8080
<h2>test docker</h2>


