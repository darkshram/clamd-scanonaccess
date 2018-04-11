ClamAV configuration for filesystem scanning on demand.
=====
Author of this document: Joel Barrios
Email: darksrham on gmail.com

Requirements:

  - Linux Kernel> = 3.8 compiled with CONFIG_FANOTIFY = y and CONFIG_FANOTIFY_ACCESS_PERMISSIONS = y
  - ClamAV> = 0.99.0

1) In the ClamAV configuration, one or more directories to be monitored will be defined (OnAccessIncludePath). It can be used to monitor shared directories through Samba or simply monitor /home or /home/user/Downloads.

2) Install clamav-scanner-systemd and clamav-server-systemd and clamav-update packages.

```
yum -y install clamav-scanner-systemd and clamav-server-systemd clamav-update
```

3) Three files are required: a tmpfile (clamd-scanonaccess-tmpfiles.conf), a systemd unit file (clamd@scanonaccess.service) and configuration file (scanonaccess.conf.systemd). Install each one to its respective places.

```
install -m 0644 clamd@scanonaccess.service /usr/lib/systemd/system/name.service

install -m 0644 scanonaccess.conf.systemd /etc/clamd.d/scanonaccess.conf

install -m 0644 clamd.scanonaccess-tmpfiles.conf /usr/lib/tmpfiles.d/clamd-scanonaccess.conf
```

4) Edit /etc/clamd.d/scanonaccess.conf and define the directories to be monitored (OnAccessIncludePath, you can define several lines with OnAccessIncludePath). It's imperative root is used as user for clamd.

5) Create directory /run/clamd.scanonaccess (the tmpfile will take care of doing it during next boot):

```
mkdir -m 0710 /run/clamd.scanonaccess
chgrp clamupdate /run/clamd.scanonaccess
```

6) Update units in systemd, enable and start service:

```
systemctl daemon-reload
systemctl enable clamd @ scanonaccess
systemctl start clamd @ scanonaccess
```

7) Activity record will be in /var/log/clamd-scanonaccess.log. Leave open with tail -f to perform tests.

```
tail -f /var/log/clamd-scanonaccess.log
```

8) Test the service trying to upload through Samba files with viruses while looking at the log file. Infected files will be uploaded as empty files.

Source: https://blog.clamav.net/2016/03/configuring-on-access-scanning-in-clamav.html

