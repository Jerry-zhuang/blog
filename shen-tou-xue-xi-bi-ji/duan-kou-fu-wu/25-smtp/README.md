# 25 - SMTP

## 基本信息

**简单邮件传输协议（SMTP）** 是 TCP/IP 套件中用于**发送和接收电子邮件**的协议。由于 SMTP 在接收方端限制消息排队的能力，通常会与 **POP3 或 IMAP** 一起使用。这些附加协议使用户能够将消息存储在服务器邮箱中，并定期下载它们。

在实践中，**电子邮件程序**通常使用 **SMTP 发送电子邮件**，同时使用 **POP3 或 IMAP 接收**它们。在基于 Unix 的系统上，**sendmail** 是最常用于电子邮件目的的 SMTP 服务器。商业软件包 Sendmail 包含一个 POP3 服务器。此外，**Microsoft Exchange** 提供 SMTP 服务器，并提供包括 POP3 支持的选项。

**默认端口：** 25,465(ssl),587(ssl)

## 基本操作

### 基本连接

```bash
nc -vb <IP> 25
```

### 指令枚举

```bash
nmap -p25 --script smtp-commands <IP>
```

### NTLM认证 - 信息泄露

```bash
root@kali: telnet example.com 587
220 example.com SMTP Server Banner
>> HELO
250 example.com Hello [x.x.x.x]
>> AUTH NTLM 334
NTLM supported
>> TlRMTVNTUAABAAAAB4IIAAAAAAAAAAAAAAAAAAAAAAA=
334 TlRMTVNTUAACAAAACgAKADgAAAAFgooCBqqVKFrKPCMAAAAAAAAAAEgASABCAAAABgOAJQAAAA9JAEkAUwAwADEAAgAKAEkASQBTADAAMQABAAoASQBJAFMAMAAxAAQACgBJAEkAUwAwADEAAwAKAEkASQBTADAAMQAHAAgAHwMI0VPy1QEAAAAA
```



## 命令

### 在 Linux 中发送电子邮件

```
swaks --to <email> --from test@sneakymailer.htb --header "Subject: test" --body "please click here http://10.10.14.42/" --server 10.10.10.197
```

