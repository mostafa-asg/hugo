---
title: "setcap Linux Command"
date: 2018-03-19T13:41:44+03:30
draft: false
---
Today I learned something new that I want to share with you. I knew that listening on port below 1024
requires special privilege, and to accomplish that you must be **sudoers**. But running applications with
sudo is not a perfect way because that way the application can do almost anything to your operating
system resulting unexpected results (you surely don't want the application delete all your files by **rm -rf**).  

Imagine that you have written a Golang http server and you want it to listen to port 80. To give
this privilege **only** you can use **setcap** command on unix/linux systems:
```
sudo setcap cap_net_bind_service=+ep /your/executable/file/path
```
That way you have given only one privilege to your executable and nothing more, ensuring no dangerous thing will be happen.
```
cap_net_bind_service
```
is one of the many capabilities that you can use. For seeing other capabilities type :
```
man capabilities
```
That's it :)


