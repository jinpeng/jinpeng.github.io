---
title: Install WebRTC/AppRTC Server
date: 2016-09-29 17:30:31
tags:
---

Environment:
============
Aliyun HK ECS instance, Ubuntu 14.04, Golang, NodeJS installed.  
Us domain mydomain.com, IP address xxx.xxx.xxx.xxx.

Install TURN (Traversal Using Relays around NAT server)
===================================

```bash
$ cd ~/webrtc
$ wget http://turnserver.open-sys.org/downloads/v4.4.5.3/turnserver-4.4.5.3-debian-wheezy-ubuntu-mint-x86-64bits.tar.gz
$ tar xvfz turnserver-4.4.5.3-debian-wheezy-ubuntu-mint-x86-64bits.tar.gz
$ sudo apt-get update
$ sudo apt-get install gdebi-core
$ sudo gdebi coturn*.deb
$ vim /etc/default/coturn
```
Uncomment the following line:

```shell
TURNSERVER_ENABLED=1
```

```bash
$ sudo openssl req -x509 -newkey rsa:2048 -keyout   /etc/turn_server_pkey.pem -out /etc/turn_server_cert.pem -days 99999 -nodes
$ vim /etc/turnserver.conf
```

Set the following parameters:

```conf
listening-device=eth0
listening-port=3478
listening-ip=xxx.xxx.xxx.xxx
listening-ip=xx.xx.xx.xx
stale-nonce
cert=/etc/turn_server_cert.pem
pkey=/etc/turn_server_pkey.pem
no-stdout-log
log-file=/var/tmp/turn.log
```

```bash
$ service coturn start
```

Install Collider (Signalling Server)
==========================

```bash
$ mkdir ~/webrtc
$ cd ~/webrtc
$ git clone --depth=1 https://github.com/webrtc/apprtc.git
$ ln -s `pwd`/apprtc/src/collider/collider $GOPATH/src
$ ln -s `pwd`/apprtc/src/collider/collidermain $GOPATH/src
$ ln -s `pwd`/apprtc/src/collider/collidertest $GOPATH/src
$ go get collidermain
$ go install collidermain
$ go test collider
$ nohup collidermain -port=8089 -room-server=https://mydomain.com &
```

Install AppRTC (Room Server)
========================

```bash
$ cd apprtc
$ sudo npm -g install grunt-cli
$ sudo apt-get install closure-compiler
$ sudo apt-get install python-webtest
$ vim package.json
```

change:

```json
"grunt-closurecompiler": ">=0.0.21",
```

to:

```json
"google-closure-compiler": "latest",
```

```bash
$ npm install
$ vim Gruntfile.js
```
Replace original closurecompiler with google-closure-compiler

```javascript
module.exports = function(grunt) {
  require('google-closure-compiler').grunt(grunt);

...

    'closure-compiler': {

...

  // grunt.loadNpmTasks('grunt-closurecompiler');

...

  grunt.registerTask('jstests', ['shell:genJsEnums', 'closure-compiler:debug', 'grunt-chrome-build', 'jstdPhantom']);
  // buildAppEnginePackage must be done before closurecompiler since buildAppEnginePackage resets out/app_engine.
  grunt.registerTask('build', ['shell:buildAppEnginePackage', 'shell:genJsEnums', 'closure-compiler:debug', 'grunt-chrome-build']);
```

```bash
$ grunt build
$ vim out/app_engine/constants.py
```

```python
TURN_BASE_URL = 'https://mydomain.com'

…

ICE_SERVER_BASE_URL = ''

...

WSS_INSTANCES = [{
    WSS_INSTANCE_HOST_KEY: 'mydomain.com:8089',
    WSS_INSTANCE_NAME_KEY: 'wsserver-std',
    WSS_INSTANCE_ZONE_KEY: 'us-central1-a'
}, {
    WSS_INSTANCE_HOST_KEY: 'mydomain.com:8089',
    WSS_INSTANCE_NAME_KEY: 'wsserver-std-2',
    WSS_INSTANCE_ZONE_KEY: 'us-central1-f'
}]
```

```bash
$ vim out/app_engine/apprtc.py
```
Since it does not run on GAE, remove the code to retrieve address from memcache.

```python
def get_wss_parameters(request):
  wss_host_port_pair = request.get('wshpp')
  wss_tls = request.get('wstls')

  if not wss_host_port_pair:
    wss_host_port_pair = constants.WSS_HOST_PORT_PAIRS[0]
```

```bash
$ cd ~/webrtc
$ wget https://storage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.40.zip
$ sudo apt-get install unzip
$ unzip google_appengine_1.9.40.zip
$ echo “export PATH=$PATH:~/webrtc/google_appengine” >> ~/.bashrc
$ source ~/.bashrc
$ cd ~/webrtc/apprtc
$ nohup dev_appserver.py --host=0.0.0.0 ./out/app_engine &
```

Install Nginx
===========

```bash
$ sudo apt-get install nginx
$ service nginx stop
$ cd ~
$ git clone git clone https://github.com/letsencrypt/letsencrypt
$ cd letsencrypt/
$ ./letsencrypt-auto certonly -d mydomain.com -d www.mydomain.com
$ cd /opt
$ mkdir dhparam
$ cd dhparam
$ mkdir keys
$ cd keys
$ openssl dhparam -out dhparams.pem 2048
$ cd ..
$ sudo chmod 700 keys
$ vim /etc/nginx/conf.d/mydomain.com.conf
```

```conf
upstream room {
  server 127.0.0.1:8080 ;
}

upstream collider {
  server 127.0.0.1:8089 ;
}

upstream turn {
  server 127.0.0.1:3033 ;
}

server {
  listen 80;
  server_name www.mydomain.com;
  rewrite ^(.*)$ https://$host$1 permanent;
}

server {
  server_name mydomain.com;
  return 301 https://www.mydomain.com$request_uri;
}

server {
  listen 443 ssl;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
  ssl_prefer_server_ciphers on;
  ssl_session_cache shared:SSL:10m;
  ssl_dhparam /opt/dhparam/keys/dhparams.pem;

  ssl_certificate /etc/letsencrypt/live/mydomain.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/mydomain.com/privkey.pem;
  server_name www.mydomain.com;

  location /collider {
    proxy_pass http://collider;
  }

  location /turn {
    add_header 'Access-Control-Allow-Origin' '*';
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
    proxy_pass http://turn;
    proxy_set_header Host $host;
  }

  location / {
    proxy_pass http://room;
    proxy_set_header Host $host;
  }
}
```

```bash
$ service nginx restart
```


