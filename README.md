#### Gunicorn loyhamiz uchun ishlayotganini tekshirishimiz kerak.
- ``(venv) root@test-server:~/uzum_drf# gunicorn --bind 0.0.0.0:8000 loyhamiz_nomi.wsgi``
- Agar test muffaciyatli ishlasa ``CTRL`` + ``X`` orqali testni toxtatamiz.- 
- ``(venv) root@test-server:~/uzum_drf# deactivate``
#### Socket file yozishimiz shart Gunicorn uchun
###### Yani biz Gunicornni django loyhamiz bilan test qilib ko'rdik endi biz django loyhamiz server bilan ishlashi va toxtashi uchun yana ishonchli usuldan foydalanishimiz kerak yani systemd va socket dan.

```angular2html
sudo nano /etc/systemd/system/gunicorn.socket
```
filni ochganimizdan so'ng shu code o'zgarishsiz yozilishi kerak socket filega.
```angular2html
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

###### Biz endi systemd service yozishimiz kerak Gunicorn uchun 
```angular2html
sudo nano /etc/systemd/system/gunicorn.service
```
```angular2html
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/root/myprojectdir
ExecStart=/root//myprojectdir/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          root.wsgi:application

[Install]
WantedBy=multi-user.target
```
###### Enable qilishimiz shart /run/gunicorn.sock file yaratish uchun
```angular2html
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```

###### Gunicornning status(holat)ni tekshirib ko'rsak bo'ladi
```angular2html
sudo systemctl status gunicorn.socket
```

Natija: 
```angular2html
● gunicorn.socket - gunicorn socket
     Loaded: loaded (/etc/systemd/system/gunicorn.socket; enabled; vendor preset: enabled)
     Active: active (listening) since Mon 2022-04-18 17:53:25 UTC; 5s ago
   Triggers: ● gunicorn.service
     Listen: /run/gunicorn.sock (Stream)
     CGroup: /system.slice/gunicorn.socket

Apr 18 17:53:25 django systemd[1]: Listening on gunicorn socket.
```

file /run/gunicorn.sockni yaratganini tekshirib ko'rsak bo'ladi.
```angular2html
file /run/gunicorn.sock
```
Natija: 
```
/run/gunicorn.sock: socket
```

Gunicornning Holatini tekshirishimiz kerak: 
```angular2html
sudo systemctl status gunicorn
```
Natija:
```angular2html
● gunicorn.service - gunicorn daemon
     Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-04-18 17:54:49 UTC; 5s ago
TriggeredBy: ● gunicorn.socket
   Main PID: 102674 (gunicorn)
      Tasks: 4 (limit: 4665)
     Memory: 94.2M
        CPU: 885ms
     CGroup: /system.slice/gunicorn.service
             ├─102674 /root/myprojectdir/venv/bin/python3 /root/myprojectdir/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application
             ├─102675 /root/myprojectdir/venv/bin/python3 /root/myprojectdir/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application
             ├─102676 /root/myprojectdir/venv/bin/python3 /root/myprojectdir/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application
             └─102677 /root/myprojectdir/venv/bin/python3 /root/myprojectdir/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application

Sep 18 17:54:49 django systemd[1]: Started gunicorn daemon.
Sep 15 17:10 n:49 django gunicorn[102674]: [2022-04-18 17:54:49 +0000] [102674] [INFO] Starting gunicorn 20.1.0
Sep 15 17:10:49 django gunicorn[102674]: [2022-04-18 17:54:49 +0000] [102674] [INFO] Listening at: unix:/run/gunicorn.sock (102674)
Sep 15 17:10:49 django gunicorn[102674]: [2022-04-18 17:54:49 +0000] [102674] [INFO] Using worker: sync
Sep 15 17:10:49 django gunicorn[102675]: [2022-04-18 17:54:49 +0000] [102675] [INFO] Booting worker with pid: 102675
Sep 15 17:10:49 django gunicorn[102676]: [2022-04-18 17:54:49 +0000] [102676] [INFO] Booting worker with pid: 102676
Sep 15 17:10:50 django gunicorn[102677]: [2022-04-18 17:54:50 +0000] [102677] [INFO] Booting worker with pid: 102677
Sep 15 17:10:50 django gunicorn[102675]:  - - [18/Apr/2022:17:54:50 +0000] "GET / HTTP/1.1" 200 10697 "-" "curl/7.81.0"
```

Gunicron ni qayta ishlastishimiz kerak:
```angular2html
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```