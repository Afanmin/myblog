---
title: hexo静态网站搭建及用nginx配置静态网页
date: 2020-08-11 14:56:54
tags: 
        - hexo
        - nginx
---
在搭建完成gitbook之后一段时间里，我逐渐发觉gitbook虽然能很好的支持markdown格式的文章展示，但是作为一个个人博客，它的功能过于单一。这是某个论坛网页最下角的“powered by Hexo”引起了我的注意。

### hexo简介
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页([官方文档](https://hexo.io/zh-cn/docs/))。关键词：“支持markdown”、“部署方便”，nice。

### 安装hexo
- hexo依赖Nodejs和Git所以安装他们先(ubuntu下如果没有root权限会报错)：
```bash
$ sudo apt-get install nodejs
$ sudo apt-get install npm
$ sudo apt-get install git-core
```
- 使用npm安装Hexo
```bash
sudo npm install -g hexo-cli
```

### 建立网站
在你觉得合适的路径下执行以下命令，hexo会进行网站的初始化
```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
命令执行完后，会在执行前的路径下的`<folder>`文件夹下创建一系列文件。
接着你可以尝试执行以下命令，启动服务。
```bash
$ hexo s
```
如果一切正常，你会看到以下输出：
![start server](https://res.cloudinary.com/afan1996/image/upload/v1597143448/hexo_test_ud58su.png)
然后在浏览器里输入`http://localhost:4000`，就可以看到初始建立的页面：
![start page](https://res.cloudinary.com/afan1996/image/upload/v1597143439/hexo_defaut_bgd6dz.png)

### 添加帖子
如果能在浏览器里看到Hello World的欢迎页，就说明一切顺利，接下来可以添加帖子了。相较于gitbook，hexo添加帖子的逻辑更为清晰，不需要你去考虑文件夹的布局。在网页根目录下，输入：
```bash
$ hexo new <post name>#这里的post name 就是你给帖子取的名字，值得一题的一点是，这里的post name会和生成网页的url有关，最好不要弄的太长，不然分享的链接会比较丑。
```
在没有对配置做任何更改的前提下，在执行了这条命令之后，hexo会在网页目录里的source下的_posts目录下新建一个`<post name>.md`的文件。
在这个文件刚生成的时候会有以下记录：
```yml
---
title: #你设置的<post name>
date: #新建post的时间
tags: 
---
```
修改title后面的文字为你想要的标题，然后在---的下方开始写你的markdown（写完别忘了保存TaT）。

### 在帖子中插入图片
在hexo中建议的插入图片的方法有三种，我这里直讲我觉得最方便的方式，在source文件下新建一个images的文件夹，把图片放到这个文件夹里，然后markdown里插入图片的路径就写成`/images/<文件名>`

### 选择你喜欢的主题
当然hexo最好的地方我觉得是它丰富的[主题](https://hexo.io/themes/)，找到到你喜欢的主题的github，然后在网页根目录下的输入：
```bash
$ git clone <主题的git> themes/<这里写主题的名字>
```
接着修改网页根目录下的`_connfig.yml`文件，修改下方内容：
```yml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: <选择的主题>
```
接着重新执行`hexo s`你就会看到你的网站变成了你想要的样子。

### 生成静态网站
如果是部署在服务器上，通过`hexo s -p <your port>`的方式就可以把你的网站放在某个端口上展示了。但是这样做不安全，也无法支持较大量的访问。所以稳妥的方式是生成静态网页，输入：
```bash
$ hexo generate
```
你会发现网页根目录下出现了一个叫public的文件夹，里面就是生成的静态网页。生成的静态网页就可以通过apache或者nginx这样的web服务器转发了。由于在部署hexo之前，我用nginx部署了一个Django的测试网站，既然服务已经启了我就在这个基础上加一层路由就行了。

### nginx 静态网页部署
首先nginx的版本比较多，不同地方看到的差别比较大，我这里记录的是`nginx/1.14.0 (Ubuntu)`。
由于云服务器上自带nginx所以本文略过安装部分，切记不要按照一些国内的教程网站操作，他们大多用的远古版本。
如果服务器上还装有apache可能启动中的apache会占用80端口可以通过以下命令查看端口是否被占用：
```bash
$ sudo lsof -iTCP:80
```
如果有输出，并且comment下的不是nginx，就用`kill -9 <PID>`把他们杀死。
接着配置nginx，进入`/etc/nginx/sites-available`输入`sudo vim default`编辑default文件。
将以下配置写入文件：
```conf
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name _;

        location / {
                alias /var/www/<yourpath>/;#这里推荐/var/www/下的目录，如果用其他目录会有权限的文图导致网页报404的错。
                index  index.html index.htm;
                autoindex on;
        }
}
```
之后运行命令：`sudo nginx -s restart`
结下来还有一步，之前`hexo generate`生成的静态网页是在网页根目录下的public里，我们需要把它们移到nginx配置里的/var/www下的目录中去。为了方便之后的网页更新，我们可以把hexo的输出静态文件目录改为/var/www下的目录，修改网页根目录下的_config.yml中的以下内容：
```yml
# Directory
source_dir: source
public_dir: /var/www/<yourpath> #把这里修改成nginx设置下alias后面的地址。
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:
```
接着运行`sudo hexo generate`（注意这里需要sudo），接下来，直接在浏览器里输入你的域名，你就能看到自己建立的网站了。
后续每次添加新的post后，运行`sudo hexo generate`便可对网站进行更新。