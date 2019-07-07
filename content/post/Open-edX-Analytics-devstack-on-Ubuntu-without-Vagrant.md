---
title: Open edX Analytics devstack on Ubuntu without Vagrant
date: 2016-04-28 16:32:04
tags:
---
## Open edX Analytics devstack on Ubuntu without Vagrant##

```bash
$ ssh root@123.56.40.252
# adduser ubuntu
# chmod +w /etc/sudoers
# vim /etc/sudoers
```

Add following line into the file:

```
ubuntu  ALL=(ALL:ALL) ALL
```
Make it looks like:

```
...
# User privilege specification
root    ALL=(ALL:ALL) ALL
ubuntu  ALL=(ALL:ALL) ALL
...
```
```
# chmod -w /etc/sudoers
# su - ubuntu
$ git clone https://github.com/edx/edx-analytics-pipeline
$ cd edx-analytics-pipeline
$ source venv/bin/activate
(venv)$ make bootstrap
```