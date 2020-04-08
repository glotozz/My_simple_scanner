### MyPortScan

参考一些网上资料自己写的一个端口扫描器

#### 一、处理命令行，选择`argparse`

```
import argparse


parser = argparse.ArgumentParser(description='MyPortScan')
parser.add_argument('--host', required=False)
parser.add_argument('--port', required=False, default='21,22,80,139,143,445,3306,5001,8080,9000')
args = parser.parse_args()
```

#### 二、解析host

```
import socket


socket.gethostbyname(hostname)
```

#### 三、端口请求

```
conn = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # 建立TCP的套接字
conn.connect((ip, port))  # 连接IP地址和对应的端口
conn.send('gqy_test\r\n'.encode('utf-8'))  # 发送垃圾数据
result = conn.recv(100)  # 设置接收数据的容量
```

#### 四、多线程+信号量

> 信号量，当`acquire()`函数被调用时，计数器-1，当`release()`函数被调用时，计数器+1，当计数器为0时，`acquire()`将阻塞线程直到其他线程调用`release()`

```
t = threading.Thread(target=try_conn, args=(ip, port))
t.start()
```

#### 五、完整代码

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# @Time    : 2020/4/8 10:04
# @Author  : glotozz
import argparse
import socket
import threading


def try_conn(ip, port):
    try:
        conn = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # 建立TCP的套接字
        conn.connect((ip, port))  # 连接IP地址和对应的端口
        conn.send('gqy_test\r\n'.encode('utf-8'))  # 发送垃圾数据
        result = conn.recv(100)  # 设置接收数据的容量
        screenLock.acquire()  # 加锁，使其他线程无法进入线程池
        print('[+]%d/tcp open' % port)  # 如果没有出错，打印端口开放，并把接收到的banner信息打印出来
        if str(result):
            print('[ %s banner]' % (str(result)))
    except Exception as e:
        screenLock.acquire()  # 解锁
        # print(e)
        print('[-]%d/tcp closed' % port)
    finally:
        screenLock.release()
        conn.close()


class Exp(object):
    def __init__(self, host, port):
        self.host = host
        self.port = port

    def run(self):
        try:
            ip = socket.gethostbyname(self.host)
        except:
            print('[-]请检查域名')
            return

        socket.setdefaulttimeout(1)
        for port in self.port:
            t = threading.Thread(target=try_conn, args=(ip, port))
            t.start()


parser = argparse.ArgumentParser(description='MyPortScan')
parser.add_argument('--host', required=False, default='127.0.0.1')
parser.add_argument('--port', required=False, default='21,22,80,139,143,445,3306,5001,8080,9000')
args = parser.parse_args()
port = list(map(int, args.port.split(',')))
e = Exp(args.host, port)
print(e.port)
# 信号量，当acquire()函数被调用时，计数器-1，当release()函数被调用时，计数器+1，当计数器为0时，acquire()将阻塞线程直到其他线程调用release()
screenLock = threading.Semaphore(value=1)
e.run()

```

#### 六、测试截图

![Snipaste_2020-04-08_10-26-21.png](https://i.loli.net/2020/04/08/WRlzJAf1YZKdOaM.png)