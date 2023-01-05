---
title: github repo:Shellcrypt
date: 2023-01-05 15:16:53
tags: [Github, SecTool]
---

# 来源

https://github.com/iilegacyyii/Shellcrypt

对ShellCode进行指定加密算法和秘钥进行加密混淆，并按照指定格式输出；

用这东西不仅需要python环境，还需要导入一些库，为什么不直接用msfvenom？

# 解析

## 加密

除了XOR使用自循环异或外，其他五个都用了python Crypto库；

```py
from Crypto.Cipher import AES, ARC4, ChaCha20, Salsa20
```

> xor

```python
return bytearray(a ^ b for (a, b) in zip(plaintext, cycle(self.key)))
```

> aes_128

```py
aes_cipher = AES.new(self.key, AES.MODE_CBC, self.nonce)
plaintext = pad(plaintext, 16)
return bytearray(aes_cipher.encrypt(plaintext))
```

> rc4

```py
rc4_cipher = ARC4.new(self.key)
return rc4_cipher.encrypt(plaintext)
```

> chacha20

```py
chacha20_cipher = ChaCha20.new(key=self.key)
return chacha20_cipher.encrypt(plaintext) 
```

> salsa20

```py
salsa20_cipher = Salsa20.new(key=key)
return salsa20_cipher.encrypt(plaintext)
```

## 输出

指定输出改变的不是加密后的SHELLCODE本身，只是变量格式，按照指定编程语言的变量声明方式输出出来而已 ：

{% asset_img 2023-01-05-15-31-52-image.png %}

# 学习

1. 几种加密算法和py实现方式；

2. 没用过的一种打印表达式；
   
   ```py
   w = 2
   print('%.2f' %w)
   print(f'w = {w:.2f}')
   ```

3. 动态执行函数的另外一种方式；
   
   之前用过eval：
   
   ```py
   def exec_something():
       pass
   
   dynamic_method = "exec_somthing"
   eval(dynamic_method+'()')
   ```
   
   这次学习到还可以这样用：
   
   ```py
   def __init__(self):
       self.__handlers = {
       "method1": self.test_method1,
       "method2": self.test_method2
   }
   def test_method1():
       pass
   def test_method2():
       pass
   
   #call
   method = "method2"
   self.__handlers[method]()
   ```
   
   

4. 简单if/else判断的单行使用方式；
   
   ```py
   num = 1
   print(num if num > 0 else num++)
   ```

5. 简单for循环的单行使用方式；
   
   ```py
   array = [1, 2, 3, 4, 5]
   print(num++ for num in array)
   ```
