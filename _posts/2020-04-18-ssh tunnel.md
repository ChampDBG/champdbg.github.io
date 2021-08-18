---
layout: post
title: "ssh tunnel"
date: 2020-04-18 15:00:00 +0900
categories: [program, daily]
tags: [proxy, ssh tunnel]
---

最近中職領先全球開季，但因為版權的關係，所以國外的 IP 沒辦法收看 twitch 的轉播。想說用 VPN 就可以解決，沒想到宿舍提供的網路阻擋 VPN 的連線~~(機車)~~。原本在 JF 實驗室留的 shadowsocks 連線，也因為 server 的硬體掛掉不能使用。

幸好，在 Z.F.Chaing 大師的介紹下，改用 ssh tunnel 的方式進行連線，簡單紀錄一下怎麼處理。

## 事前準備
* 當作跳版的 server

* 建立 ssh 連線的工具
  
  * Windows 用戶可能是 [PuTTY](https://www.putty.org), [MobaXterm](https://mobaxterm.mobatek.net/), 或 [WSL](https://blog.miniasp.com/post/2019/02/01/Useful-tool-WSL-Windows-Subsystem-for-Linux)，我自己是用 WSL。
  
  * Linux 用戶使用 bash 的 ssh 即可

## 實現方法
### 建立連線
先打開 wsl 後輸入這段指令
```
$ ssh -D 8080 user@my.host.org
```
> * user 輸入使用者名字
>
> * my.host.ort 輸入 server IP

如果有 port 的話再自己加上去，例如
```
$ ssh -D 8080 -p 9527 user@my.host.org
```
成功登入之後，就把連線放著，可以搭配 tmux 之類的東西放在背景執行。

### 設定網路 
1. 打開「網際網路選項」，選擇「連線」、「LAN 設定」(拍謝，我只能提供日文圖片，但應該沒什麼差)
   
   ![](/img/20200417_internet_option.png)


2. 接著打勾「自動偵測設定」、「為您的 LAN 使用 Proxy 伺服器...」、「進階」
   
   ![](/img/20200417_internet_option_2.png)

3. 輸入 socks 的內容
   
   ![](/img/20200417_internet_option3.png)

設定完成之後，只要打開瀏覽器(Chrome, Firefox) 即可使用。

## Reference
[1] SSH Tunnel 通道打造加密 Proxy，透過外部 Linux 伺服器上網 [[link](https://blog.gtwang.org/linux/ssh-tunnel-socks-proxy-forwarding-tutorial/)]

[2] 利用 SSH Tunnel 做跳板（aka. 翻牆）[[link](https://blog.rex-tsou.com/2017/10/%E5%88%A9%E7%94%A8-ssh-tunnel-%E5%81%9A%E8%B7%B3%E6%9D%BFaka.-%E7%BF%BB%E7%89%86/)]
