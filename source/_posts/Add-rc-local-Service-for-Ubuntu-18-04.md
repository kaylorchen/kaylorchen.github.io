---
title: Add rc.local Service for Ubuntu 18.04
comments: true
date: 2019-06-25 17:12:15
tags:
- Ubuntu
categories:
- Linux
---


# Modify ***/lib/systemd/system/rc-local.service***

Append some content:
```
[Unit]
Description=/etc/rc.local Compatibility
Documentation=man:systemd-rc-local-generator(8)
ConditionFileIsExecutable=/etc/rc.local
After=network.target

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
RemainAfterExit=yes
GuessMainPID=no

[Install]
WantedBy=multi-user.target
Alias=rc-local.service
```

# Configure ***/etc/rc.local***
```bash
touch /etc/rc.local
chmod 755 /etc/rc.local
```
Edit ***/etc/rc.local***, and the content is as follows:
```
#!/bin/bash
echo "test rc " > /var/test.log
```

Reboot your OS, and check ***/var/test.log***.