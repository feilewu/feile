---
title: openwrt通过frp代理后外网无法访问
id: c1724c50-2a87-47d7-bfce-e3d32fc987a4
date: 2024-10-01 13:40:29
auther: admin
cover: 
excerpt: 外网（frp）访问openwrt，报错request entity too large 错误原因：通常是指请求的数量超过服务器的限制 解决方法：修改配置文件，登录到opnwrt的ssh页面输入 vi /etc/config/uhttpd 修改 option max_requests '50'改成5
permalink: /?p=c1724c50-2a87-47d7-bfce-e3d32fc987a4
categories:
tags: 
 - openwrt
---



外网（frp）访问openwrt，报错request entity too large

![](https://pic.feilewu.cn/uploads/2024/10/01/a403621c-a246-414c-bfca-c604a8153ff1.png)

错误原因：通常是指请求的数量超过服务器的限制

解决方法：修改配置文件，登录到opnwrt的ssh页面输入

```
vi /etc/config/uhttpd
```

修改 `option max_requests '50'`改成500

`option max_requests '50'`的意思是设置最大请求数为500。

![](https://pic.feilewu.cn/uploads/2024/10/01/922be141-442e-4f81-90be-078131a1997f.png)



重启

切换浏览器