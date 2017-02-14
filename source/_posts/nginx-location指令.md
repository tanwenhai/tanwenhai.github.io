---
title: nginx location指令
date: 2017-02-14 15:50:11
tags: nginx
---
```nginxy
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

* configuration A 精确匹配 path=/
* configuration B 匹配 path=/* 如 /index.html
* configuration C 匹配 path=/documents/* 如 /documents/document.html
* configuration D 正则唯一匹配 path=/images/* 如 /images/1.gif
* configuration E 正则不区分大小写匹配 *.(gif|jpg|jpeg) 如 /documents/1.jpg


