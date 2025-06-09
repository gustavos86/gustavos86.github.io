---
title: Systemd example
date: 2025-06-09 16:00:00 -0700
categories: [LINUX, SYSTEMD]
tags: [linux, systemd]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

Here an example of how to make OpenVPN iniate as a Service using Systemd.

```
root@openvpn-server:~# cat /etc/systemd/system/openvpn.service
[Unit]
Description=openvpn
Wants=network-online.target test-internet.service
After=network-online.target test-internet.service

[Service]
User=root
SyslogIdentifier=openvpn
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/sbin/openvpn --config /home/root/openvpn-server.ovpn
Restart=always

[Install]
WantedBy=multi-user.target
root@openvpn-server:~# 
```

Enable the service

```
root@openvpn-server:~# systemctl status openvpn

root@openvpn-server:~# systemctl start openvpn

root@openvpn-server:~# systemctl status openvpn

root@openvpn-server:~# systemctl enable openvpn
```