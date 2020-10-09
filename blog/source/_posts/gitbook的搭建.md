---
title: 基于gitbook的博客搭建
date: 2020-08-11 10:03:07 
tags: web
category: gitbook
---
## 在腾讯云轻量级服务器上部署 gitbook

在服务器上部署了SSR之后，我发现服务器可怜的1G内存竟然还有700多m的剩余，反正也注册了域名，就打算搭一个博客看看，想了一下，从头学后端太麻烦，想起之前在自己电脑弄过gitbook，部署起来也很简单，作为博客功能也基本都能满足，就试试看在服务器上部署gitbook，事实上确实很简单。

#### 1. 安装nodejs和npm 

```bash
$ sudo apt install nodejs
$ sudo apt install npm
```
#### 2. 安装gitbook

这里在ubuntu里需要root权限。

```bash
$ sudo npm install gitbook-cli -g
```

#### 3. 创建一本书

```bash
$ gitbook init
```

创建一个新文件夹，在这个文件夹里执行完这条语句之后，文件夹里会生成两个文件`README.md`、`SUMMARY.md`。

在`SUMMARY.md`下存储了本书的逻辑目录，格式如下：

```
# Summary
* [Introduction](README.md)
  * [二级目录](README.md)
```

#### 4. 启动服务

在`xxxx`端口启动服务，这样就可以在本地访问gitbook页面。我懒得弄后端，就直接在服务器上启动了一个本地服务，并且把那个端口对外暴露，然后就可以在网络上访问到了。

```bash
$ gitbook serve --port=xxxx
```

#### 5. 生成静态页面

当然比较安全的方式是生成静态的页面把它放在后端，通过一定的路由方式来访问。

```bash
$ gitbook build
```



#### Tips. 自动生成SUMMARY.md

手动去写`SUMMARY.md`很烦，所以我写了一个自动脚本可以在文件夹下根据文件夹内的`.md`文件自动生成`SUMMARY.md`。

这个也没啥好说的。

```python
def generateIndex(dir,depth,root=False,filename = 'SUMMARY.md'):
    def myWalkThrough(dir,depth,fp,filename,root=False):
        #print(filename)
        ls = os.listdir(dir)
        ls.sort()
        readme = 'README.md'
        if not root:
            if readme in ls:
                fp.write(depth+' '+'['+dir.split('/')[-1]+']'+'('+ dir+'/'+ readme +')\n')
                print(depth,'['+dir.split('/')[-1]+']'+'('+ dir+'/'+ readme +')')
            depth = '   ' + depth
        for item in ls:
            if os.path.isfile(dir+'/'+item):
                if os.path.splitext(item)[-1] == '.md':
                    if item == readme and not root:
                        continue
                    if root and item == filename:
                        continue
                    fp.write(depth+' '+'['+ os.path.splitext(item)[0] +']'+'('+dir+'/'+item+')\n')
                    print(depth,'['+ os.path.splitext(item)[0] +']'+'('+dir+'/'+item+')')

        for item in ls:
            if not item.startswith('.') and not item.startswith('_'):
                if os.path.isdir(dir+'/'+item):
                    myWalkThrough(dir+'/'+item,depth,fp,filename)
    with open(filename,'w') as fp:
        fp.write('# '+filename.split('.')[0]+'\n')
        myWalkThrough(dir,depth,fp,filename,root=root)
```

