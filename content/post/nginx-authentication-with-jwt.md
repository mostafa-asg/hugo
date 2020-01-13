---
title: "Configure Nginx Authentication With JWT"
date: 2020-01-13T20:35:57+03:30
draft: false
---
#### Prerequisites
Ensure your machine have the required library and tools. I tested on Ubuntu 18.
```
sudo apt update
sudo apt install libtool-bin build-essential libfuse-dev libcurl4-openssl-dev libxml2-dev mime-support automake libtool pkg-config autoconf
```
1) Download Jansson C library source code from [here](http://www.digip.org/jansson/#releases) then:
```
$ tar -xvf jansson-2.12.tar.gz
$ cd jansson-2.12/
$ ./configure
$ make
$ sudo make install
$ cd ..
```
2) We need [JWT C Library](https://github.com/benmcollins/libjwt). Make sure `autoreconf` command is installed:
```
$ git clone https://github.com/benmcollins/libjwt
$ cd libjwt
$ autoreconf -i
$ ./configure
$ make
$ sudo make install
$ cd ..
```
3) Download the source code of [Nginx](https://nginx.org/en/download.html) and extract it:
```
$ tar -xvf nginx-1.16.1.tar.gz
$ cd nginx-1.16.1
$ cd ..
```
4) Clone [Nginx JWT module](https://github.com/TeslaGov/ngx-http-auth-jwt-module):
```
$ git clone https://github.com/TeslaGov/ngx-http-auth-jwt-module
```
5) Configure Nginx to use this module:
```
$ cd nginx-1.16.1
$ ./configure --add-module=../ngx-http-auth-jwt-module --without-http_gzip_module --with-http_ssl_module
$ sudo cp /usr/local/lib/libjwt.* /lib/x86_64-linux-gnu/
$ make
$ sudo make install
```
6) Use `auth_jwt` directive in Nginx config. Read documentation on [Nginx JWT module](https://github.com/TeslaGov/ngx-http-auth-jwt-module). Here is just a sample:
```
  location / {
            auth_jwt_key "4d6f737461666120417367617269";
            auth_jwt_enabled on;
            auth_jwt_algorithm HS256; # or RS256
            auth_jwt_validate_email off;  # or off
            proxy_pass http://google.com;
        }
```
7) [Create a JWT token](https://jwt.io/#debugger-io) and put it in the `authorization` header and make a request to Nginx:
```
curl -H "authorization: Bearer {JWT}" {NGINX_SERVER}
```
8) If any problems occurred check Nginx logs.
