# SCUT fakepass搭建指南
## 基本思路
从微信进入"My Pass我的通行证"后会向服务器发出网页请求，手机上显示请求到的html页面。可以通过http手机代理将手机发出的http请求转发给一个中间人服务器，由中间人服务器转发http请求，服务器返回数据包后被中间人服务器拦截，用其他数据包代替传回客户端。从而让客户端显示特定的网页内容。fakepass自然显示的是审批通过的页面。

## 环境条件
- 一台具有公网IP的服务器，安装有linux系统
- 手机端http代理
- whistle抓包工具
- fakepass html页面及相关数据
- 以上资源除公网服务外后文都有提到获取方式

## 搭建fakepass
### 1.安装Apache服务
安装Apache服务用来存放fakepass页面及相关数据，安装Apache的步骤这里不给出，需要保证服务器防火墙以及运营商后台同时开放80号端口。将下面地址下载的数据包放在`/var/www/html/fakepass`中，其中fakepass目录需要自己创建。
```sh
# 感谢@huohuaqi
https://github.com/huohuaqi/huohuaqi.github.io.git
```
使用时请将`/var/www/html/fakepass/index.html`和`/var/www/html/fakepass/js/index_h5.js`文件中的所有`https://www.siker.top/fakepass`替换为`https://你的公网IP/fakepass`或者`https://你的域名/fakepass`, 否者界面无法正常显示。

### 2.在服务器上安装whistle抓包工具
官网有详细的安装使用教程，可以参考[官网](https://wproxy.org/whistle/install.html)
- 安装node.js
```sh
# 如果服务器内存太小是无法编译的，推荐包管理工具安装
sudo apt-get install -y nodejs
# 查看版本
node -v
```
- 安装whistle
```sh
npm install -g whistle
# 启动whistle，并指定后台登录账户和密码
w2 start -n name -w passwd
```
whistle的默认端口：8899 (如果端口被占用，可以在启动时通过 -p 来指定新的端口，更多信息可以通过执行命令行 w2 help。需要保证服务器防火墙以及运营商后台同时开放8899号端口。然后在浏览器地址栏输入`服务器公网IP:8899`，输入启动时设置的账号和密码即可登录whistle后台，如果没有设置密码请忽略。

### 3.配置手机http代理
mypass是使用加密的https协议的，需要安装证书才能抓取https数据包，首先在手机上安装证书。[安装证书请参考文档](http://wproxy.org/whistle/webui/https.html)。由于whistle代理是只支持http的，所以需要配置http代理。ios端推荐使用shadowrocket(小火箭)，安卓端推荐使用xxxx，在代理软件中填入公网服务器IP，端口号8899，上面设置的账户和密码，然后开启代理即可。

### 4.设置whistle规则
登录whistle后台，点击左边栏第二个按钮rules，创建一个新规则，名字随意。填入下面内容:
```conf
/enroll.scut.edu.cn/ 服务器公网IP:80/fakepass/index.html
www.baidu.com/apply.html 服务器公网IP:80/fakepass/apply.html
```
保存后即可。这里定制了两条规者，whistle会拦截所有中间是`/enroll.scut.edu.cn/`的请求，并用`服务器公网IP:80/fakepass/index.html`取代回应。下一条规则同理。这里之所以是百度，是因为网页中设置了点击按钮会跳转到`www.baidu.com/apply.html`，当然这个页面不存在，这个请求会被whistle拦截，用`服务器公网IP:80/fakepass/apply.html`取代。由于微信中链接跳转不能用IP地址，且只能使用备案的域名，没有备案的域名的话就只能用这种方法绕过了。
