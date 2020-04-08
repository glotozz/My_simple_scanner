### MyDirScan

参考一些网上资料自己写的一个目录扫描器

#### 一、处理命令行，选择`argparse`

```
import argparse


parser = argparse.ArgumentParser(description='MyDirScan')
parser.add_argument('--url', required=False, default='http://127.0.0.1/')
parser.add_argument('--dict', required=False, default='PHP.txt')
args = parser.parse_args()
```

#### 二、读取文件

```
dict = []
f = open(self.dict)
line = f.readline()
while line:
	dict.append(line.strip())
    line = f.readline()
```

#### 三、head请求

```
r = requests.head(url, timeout=5)
```

#### 四、多线程

```
t = threading.Thread(target=try_conn, args=(url,))
t.start()
```

#### 五、完整代码

```python
import argparse
import requests
import threading


def try_conn(url):
    r = requests.head(url, timeout=5)
    # print('[+]{}    {}'.format(url, r.status_code))
    if r.status_code != 404:
        print('[+]{}    {}'.format(url, r.status_code))


class Exp(object):
    def __init__(self, url, dict):
        self.url = url
        self.dict = dict

    def run(self):
        dict = []
        f = open(self.dict)
        line = f.readline()
        while line:
            dict.append(line.strip())
            line = f.readline()

        for dic in dict:
            url = self.url + dic
            t = threading.Thread(target=try_conn, args=(url,))
            t.start()


parser = argparse.ArgumentParser(description='MyDirScan')
parser.add_argument('--url', required=False, default='http://127.0.0.1/')
parser.add_argument('--dict', required=False, default='PHP.txt')
args = parser.parse_args()
e = Exp(args.url, args.dict)
e.run()

```

#### 六、测试截图

![Snipaste_2020-04-08_11-50-19.png](https://i.loli.net/2020/04/08/gYGlMrI1ST6iL9A.png)