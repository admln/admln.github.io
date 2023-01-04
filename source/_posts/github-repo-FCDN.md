---
title: github repo:FCDN
date: 2023-01-04 20:16:25
tags: [Github, SecTool]
---

# 来源

https://github.com/ccc-f/FCDN

批量检测域名是不是有CDN/WAF/DMZweb防护，找出疑是可以解析出真实IP的域名。

不过根据它的逻辑，与其说是找真IP域名，不如说是过滤掉那些肯定被防护的域名。

本身逻辑并不复杂。

# 原理

检测逻辑跟它的readme说的差不多，就使用dnspython模拟nslookup进行域名A记录解析，如果返回多个IP或者nameserver域名跟所查域名长度不一致即判断为CDN防护。然后剩下的就是可能是真实IP解析的域名。

```python
resolver = dns.resolver.Resolver()
resolver.nameservers = ['1.1.1.1', '8.8.8.8']
domain = domain.strip()
a = resolver.resolve(domain, 'A')
for index,value in enumerate(a.response.answer):
    for j in value.items:
        if re.search(r'\d+\.\d+\.\d+\.\d+', j.to_text()):
            ipcount += 1
            if ipcount >= 2:
                return True,domain
        elif re.search(r'(\w+\.)+', j.to_text()):
            cname = j.to_text()[:-1]
            length = len(cname)
            if length != len(domain):
                return True,domain
            else:
                return False,domain
        else:
            return False,domain
if ipcount == 1:
    return False,domain
```

其他都是输入输出和多线程代码。

# 学习

1. CDN防护确实是攻击方必然要考虑的，其相应的知识不可少；

2. 在线cdn检测网站：https://myssl.com/cdn_check.html；

3. 多线程的简单实用；

4. dnspython库的应用；



# 缺点

在它主动表示检测逻辑简单，只是有一个反其道而行之的亮点后，具体能力不谈，建议加一个扫子域名的功能。

在拿到一个站点域名，还需要先解析出主域名的各个子域名再喂给它这个脚本去跑，如果能做到给个二级域名自动扫子域名+IP解析判断会更易用些。
