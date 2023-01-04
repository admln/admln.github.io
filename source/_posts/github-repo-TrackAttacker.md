---
title: github repo:TrackAttacker
date: 2023-01-04 14:12:44
tags: [SecTool, Github]
---

# FROM

https://github.com/Bywalks/TrackAttacker

就是批量对IP进行各类所需信息的API进行调用返回有用信息，没什么技术含量。

缺点是返回信息的整理不够，实战中怕是使用困难。

# SUBJECT

## 获取IP所属地理位置

API:https://www.ip138.com/iplookup.asp

```python
url = "https://www.ip138.com/iplookup.asp?ip="+str(ip)+"&action=2"
req = requests.get(url,timeout=3,headers=headers,verify=False)
req.encoding = "gbk"
address=re.findall('"ASN归属地":"(.*?)",\s"iP段":',req.text) 
if address != "":
    print("[+]Address:"+address[0])
```

## 获取网站备案信息

API:https://www.beian88.com/home/Search

```python
url = "https://www.beian88.com/home/Search"
post_site = {'d': site}
req = requests.post(url,data=post_site,timeout=3,headers=headers,verify=False)
req.encoding = "utf-8"
key=re.findall('"key":"(.*?)"}',req.text) 
url1 = "https://www.beian88.com/d/" + key[0]
requ = requests.get(url1,timeout=3,headers=headers,verify=False)
requ.encoding = "utf-8"
name=re.findall('<span class="field-value" id="ba_Name">(.*?)</span>',requ.text)
if name[0] != "":
    #print("备案信息")
    webname=re.findall('<span class="field-value" id="ba_WebName">(.*?)</span>',requ.text) 
    print("[+]网站名称:"+webname[0]) 
    print("[+]主办单位名称:"+name[0])
    type=re.findall('<span class="field-value" id="ba_Type">(.*?)</span>',requ.text) 
    print("[+]主办单位性质:"+type[0])
    license=re.findall('<span class="field-value" id="ba_License">(.*?)</span>',requ.text) 
    print("[+]网站备案/许可证号:"+license[0])
```

## 获取IP绑定域名

API:https://site.ip138.com

```python
url = "https://site.ip138.com/"+str(ip)+"/"
req = requests.get(url,timeout=3,headers=headers,verify=False)
req.encoding = "utf-8"
site=re.findall('<li><span\sclass="date">[\d\-\s]+</span><a\shref=".*?"\starget="_blank">(.*?)</a></li>',req.text) 
if site != "":
    print("[+]Site:"+site[0])
    return site[0]
```

## 获取域名WHOIS信息

API:http://whois.4.cn/api/main

```python
url = "http://whois.4.cn/api/main"
post_site = {'domain': site}
req = requests.post(url,data=post_site,headers=headers,verify=False)
json_data = json.loads(req.text)
if json_data['data']['owner_name'] !="":
    #print("Whois信息")
    print("[+]域名所有者:"+json_data['data']['owner_name'])
    print("[+]域名所有者邮箱:"+json_data['data']['owner_email'])
    print("[+]域名所有者注册:"+json_data['data']['registrars'])
```

## NMAP扫描端口

import：nmap

```python
n = nmap.PortScanner() 
ip = "\""+ip+"\""
n.scan(hosts=ip,arguments="-sV -p 22,80,90,443,1433,1521,3306,3389,6379,7001,7002,8000,8080,9090,9043,9080,9300,50050")
for x in n.all_hosts():
    if n[x].hostname() != "":
        print("[+]HostName: " + n[x].hostname())
    for y in n[x].all_protocols():
        print("[+]Protocols: " + y)
        for z in n[x][y].keys():
            if n[x][y][z]["state"] == "open":
                print("[+]port: " + str(z) + " | name: " + n[x][y][z]["name"] + " | state: " + n[x][y][z]["state"])
```

# study

1. 对于简单快速匹配来说，re.findall函数挺好用的；

2. nmap可以直接代码集成调用，内嵌方便；
