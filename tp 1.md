### S'assurer que le service sshd est démarré

```
sudo systemctl status sshd

    Active: active(running)
```


### Analyser les processus liés au service SSH

 ```
ps ax | grep "sshd"

    710 ? Ss 0:00 sshd: /usr/sbin/sshd ...
    16929 ttyl 0:00 grep --color=auto sshd
```
###  Déterminer le port sur lequel écoute le service SSH


```
sudo ss -tlnp
State       Recive-QSend-Q  Local Address:Port  Peer Address:Port Process
LISTEN      0       128     0.0.0.0:22         0.0.0.0:*        users:(("sshd",pid=710,fd=3))
LISTEN      0       128     0.0.0.0:22         [::]:*           users:(("sshd",pid=710,fd=4))
```

###  Consulter les logs du service SSH

```
journalctl | grep "sshd"
Dec 02 03:43:28 localhost systemd[1]: Created slice Slice /system/sshd-keygen.
...
```
###  Identifier le fichier de configuration du serveur SSH
```
/etc/ssh/sshd_config
```

###  Modifier le fichier de conf
```
[me@localhost sshd]$ sudo cat ./sshd_config | grep "Port"
Port 12345



sudo firewall-cmd --list-all | grep "ports" 
ports: 12345/tcp 22/tcp
``` 

###  Redémarrer le service

```
sudo systemctl restart sshd
```

###  Effectuer une connexion SSH sur le nouveau port

```
┌─[✗]─[matyspg@parrot]─[~]
└──╼ $ssh -p12345 me@10.1.1.1
The authenticity of host '[10.1.1.1]:12345 ([10.1.1.1]:12345)' can't be established.
ED25519 key fingerprint is SHA256:1Mrk1sfMQnhE3es53dTDZH3T4mmU7KOoj/oyj753lz0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.1.1.1]:12345' (ED25519) to the list of known hosts.
me@10.1.1.1's password: 
Last login: Mon Dec  2 03:44:20 2024
```

###  Installer le serveur NGINX

```bash
[me@efrei-xmg4agau1 ~]$ sudo dnf install nginx

...


Installed:
  nginx-2:1.20.1-20.el9.0.1.x86_64                                              
  nginx-core-2:1.20.1-20.el9.0.1.x86_64                                         
  nginx-filesystem-2:1.20.1-20.el9.0.1.noarch                                   
  rocky-logos-httpd-90.15-2.el9.noarch         
  ```


  ###  Démarrer le service NGINX    

  ```bash
   me@efrei-xmg4agau1 ~]$ sudo systemctl enable nginx
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
[me@efrei-xmg4agau1 ~]$ sudo systemctl start nginx
```

###  Déterminer sur quel port tourne NGINX
```bash
[me@efrei-xmg4agau1 ~]$ sudo firewall-cmd --permanent --add-port=80/tcp
success
[me@efrei-xmg4agau1 ~]$ sudo firewall-cmd --reload
success
me@efrei-xmg4agau1 ~]$ sudo firewall-cmd --list-all | grep "ports"
  ports: 12345/tcp 22/tcp 80/tcp
```
###  Déterminer les processus liés au service NGINX

```bash
[me@efrei-xmg4agau1 ~]$ sudo ps aux | grep nginx
root        1367  0.0  0.0  11292  1592 ?        Ss   16:07   0:00 nginx: master process /usr/sbin/nginx
nginx       1368  0.0  0.2  15532  4920 ?        S    16:07   0:00 nginx: worker process
me          1599  0.0  0.1   6408  2176 pts/0    S+   16:32   0:00 grep --color=auto nginx
```

###  Déterminer le nom de l'utilisateur qui lance NGINX

```bash
[me@efrei-xmg4agau1 ~]$ cat /etc/passwd | grep nginx
nginx:x:996:993:Nginx web server:/var/lib/nginx:/sbin/nologin
```

### Test !

```
┌─[✗]─[matyspg@parrot]─[~]
└──╼ $curl http://10.1.1.1:80 | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/
      
      html {
100  7620  100  7620    0     0  4526k      0 --:--:-- --:--:-- --:--:-- 7441k
curl: Failed writing body
```

###  Déterminer le path du fichier de configuration de NGINX

```bash
[me@efrei-xmg4agau1 etc]$ ls -al /etc/nginx/nginx.conf
-rw-r--r--. 1 root root 2334 Nov  8 17:43 /etc/nginx/nginx.conf
```

###  Trouver dans le fichier de conf

```bash
me@efrei-xmg4agau1 etc]$ cat /etc/nginx/nginx.conf | grep "server {" -A 25
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

 [me@efrei-xmg4agau1 etc]$ cat /etc/nginx/nginx.conf | grep "^include"
include /usr/share/nginx/modules/*.conf;
me@efrei-xmg4agau1 etc]$ cat /etc/nginx/nginx.conf | grep "^user"
user nginx;
```
 ### Gérer les permissions
```bash
[me@efrei-xmg4agau1 tp1_parc]$ sudo chown :nginx /var/www/tp1_parc/index.html
[me@efrei-xmg4agau1 tp1_parc]$ ls -l .
total 4
-rw-r--r--. 1 root nginx 38 Dec  2 17:10 index.html
```

###  Adapter la conf NGINX
###  Visitez votre super site web

```bash
[me@efrei-xmg4agau1 tp1_parc]$ curl http://10.1.1.1:12345
<h1>MEOW mon premier serveur web</h1>
```
### Démarrer le service netdata
```bash
me@efrei-xmg4agau1 tp1_parc]$ sudo systemctl start netdata
```

###  Déterminer sur quel port tourne Netdata

```bash
[me@efrei-xmg4agau1 tp1_parc]$ sudo cat /etc/netdata/netdata.conf |grep "wget"
#  wget -O /etc/netdata/netdata.conf http://localhost:19999/netdata.conf
[me@efrei-xmg4agau1 tp1_parc]$ sudo firewall-cmd --list-all | grep ports
  ports: 12345/tcp 22/tcp 19999/tcp

```

###  Visiter l'interface Web

```bash
[me@b002-09 ~]$ curl http://10.1.1.2:19999 | head 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!doctype html><html lang="en" dir="ltr"><head><meta charset="utf-8"/><title>Netdata</title><script>const CONFIG = {
      cache: {
        agentInfo: false,
        cloudToken: true,
        agentToken: true,
      }
    }

    // STATE MANAGEMENT ======================================================================== //
    const state = {
 93  106k   93   99k    0     0  5861k      0 --:--:-- --:--:-- --:--:-- 5861k
```