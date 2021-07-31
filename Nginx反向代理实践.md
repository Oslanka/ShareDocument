## Nginx反向代理实践

### 正向代理

​		客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。

​		**简单理解正向代理：VPN，梯子，都是正向代理的实现，是对客户端的代理。**意思是对客户端加了一层代理服务器，代理服务器负责取到原始服务器的数据，并传给客户端，相当于绕了个弯路，获取到原始服务器的数据。

### 反向代理

​		反向代理（Reverse Proxy）：反向代理是指服务器根据客户端的请求,从其关系的一组或多组后端服务器(如Web服务器)上获取资源,然后再将这些资源返回给客户端,客户端只会得知反向代理的IP地址,而不知道在代理服务器后面的服务器簇的存在。

​		**简单理解反向代理：反向代理是服务端之间的代理，与客户端无关。**代理服务器打通了目标服务器与客户端，客户端无感知，这点与正向代理主要区别之一。

​		Nginx反向代理图例：

![img](https://i0.hdslb.com/bfs/album/715ad68f5a91a11a3ec546f977e3bd8801f416a2.jpg)

### 反向代理的作用

- 对客户端隐藏服务器（集群）的IP地址，隐藏目标服务器；
- 安全：作为应用层防火墙，为网站提供对基于Web的攻击行为（例如DoS/DDoS的防护，更容易排查恶意软件等；
- 为后端服务器（集群）统一提供加密和SSL加速（如SSL终端代理）；
- 负载均衡，若服务器集群中有负荷较高者，反向代理通过URL重写，根据连线请求从负荷较低者获取与所需相同的资源或备援；
- 对于静态内容及短时间内有大量访问请求的动态内容提供缓存服务；
- 对一些内容进行压缩，以节约带宽或为网络带宽不佳的网络提供服务；
- 减速上传；
- 提供HTTP访问认证。

### Nginx简介

![Nginx](https://i0.hdslb.com/bfs/album/77552569f50ea0a22a66a3dc267dfb91ebce3259.png)

​		Nginx 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务;通常用在**反向代理**、**负载均衡**和 **HTTP 缓存**。

​		作者：伊戈尔·赛索耶夫(Igor Sysoev)，Nginx之父，赛索耶夫曾就职Rambler（漫步者）COO，根据赛索耶夫本人的描述（12 年的一次采访），Nginx 是他用业余时间开发的；Igor Sysoev的老东家Rambler对Nginx提出了侵犯版权的诉讼，Rambler声称拥有所有NGINX源代码的版权，因为它是在Sysoev担任公司员工时创建的。

​		使用范围之广泛：根据 Netcraft 发布的 [最新 Web 服务器调查结果](https://news.netcraft.com/archives/2021/06/29/june-2021-web-server-survey.html) 显示，从 2019 年 4 月开始，Nginx 超过了 Apache ，成为互联网上部署最广泛的服务器，到目前2021-6月，nginx将近占有市场40%（35.98%）份额。

![Nginx1](https://i0.hdslb.com/bfs/album/b05e9c027e7c124f14c6ae76ddf2765328c1bb38.png)

### Nginx相关指令

- **listen** : 配置监听的IP/端口

```bash
listen address[:port] [default_server] [setfib=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [deferred]
    [accept_filter=filter] [bind] [ssl];
#例子
#listen *:80 | *:8080 #监听所有80端口和8080端口
#listen  IP_address:port   #监听指定的地址和端口号
#listen  IP_address     #监听指定ip地址所有端口
#listen port     #监听该端口的所有IP连接
```

- **server_name**  : 域名/IP

```bash
server_name google.com www.google.com;#可以只有一个名称，也可以有多个名称，中间用空格隔开
#使用通配符“*”
server_name *.google.com www.google.*;
#正则表达式,用“~”作为正则表达式字符串的开始标记
server_name ~^www\d+\.google\.com$;
server_name 192.168.1.1
```

- **location** : 匹配 URL,语法规则：`location [=|~|~*|^~] /uri/ { … }`
  - `=` 开头表示精确匹配
  - `^~` 开头表示uri以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格）。以xx开头
  - `~` 开头表示区分大小写的正则匹配           以xx结尾
  - `~*` 开头表示不区分大小写的正则匹配        以xx结尾
  - `!~`和`!~*`分别为区分大小写不匹配及不区分大小写不匹配 的正则
  - `/` 通用匹配，任何请求都会匹配到。

- **proxy_pass** : 被代理地址

```bash
proxy_pass  http://www.google.com;
# proxy_pass 结尾带不带/ 区别
```

- **allow**：允许访问/**deny**：拒绝访问

```bash
allow ip
deny ip
```

- **index**: 默认首页，多个空格隔开

```bash
index index.html index.htm
```

- 其他配置

```bash
keepalive_timeout 60;#超时时间
tcp_nodelay on;#防止网络阻塞
client_header_buffer_size 4k;#客户端请求头部的缓冲区大小
open_file_cache max=102400 inactive=20s;#为打开文件指定缓存，默认是没有启用
open_file_cache_valid 30s;#多长时间检查一次缓存的有效信息
open_file_cache_min_uses 1;#时间内文件的最少使用次数
client_header_timeout 15;#请求头的超时时间
client_body_timeout 15;#请求体的超时时间
reset_timedout_connection on;#告诉nginx关闭不响应的客户端连接
send_timeout 15;#响应客户端超时时间
server_tokens off;#关闭在错误页面中的nginx版本，安全性
client_max_body_size 10m;#上传文件大小限制
```

### 实践Nginx反向代理内网穿透8081端口

- 目的：隐藏8081端口，通过访问80端口实现访问8081端口。

### 实现步骤

- 我们配置api 在8081端口，并且成功部署，此时8081对外开放，所以可访问

![image-20210628230906320](https://i0.hdslb.com/bfs/album/884b514f0857166f56c398b6be64627e29e26966.png)

- 以Ubuntu环境为例

```bash
$ sudo apt-get install nginx     #安装
$ cd /etc/nginx/                 #目录
$ vim nginx.conf                 #配置
```

- Nginx http节点下 配置8081端口反向代理如下

```bash
 server {
        listen 80 default_server;
        listen [::]:80 default_server;
        location  /api/ {
                        proxy_pass http://127.0.0.1:8081;
                }

                location  /apidocs/ {  
                        proxy_pass http://localhost:8081/api/;
                        index swagger-ui.html;
                        error_page 404 http://localhost:8081/api/swagger-ui.html;
                }
         }
```

- 配置完成后重启Nginx服务

```bash
$ service nginx restart/nginx -s reload
```

- 访问http://ip/apidocs/swagger-ui.html 成功

![image-20210628230318765](https://i0.hdslb.com/bfs/album/f9f1bd30f3b47c90e281166998e966799cd04f39.png)

- 我们可以关掉服务器安全组规则，去掉8081端口-安全组规则，可以登录阿里云配置，以阿里云为例，其他同理

- http://ip/apidocs/swagger-ui.html 依然可以访问

- http://ip:8081/api/swagger-ui.html 不可访问

<img src="https://i0.hdslb.com/bfs/album/925760f9eebe5f8357f293191592edfe8e55b920.png" alt="image-20210628230803726" style="zoom:50%;" />

- 至此，我们实现了nginx反向代理8081端口，通过访问80端口，代理到8081端口的目的
- 重点理解Ngnix location&proxy_pass字段规则

### 实现方式二配置upstream

- 在http节点下，加入upstream节点

```bash
upstream demo { 
   server ip:8080; 
   server ip:8081; 
}
```

- 将server节点下的location节点中的proxy_pass配置为：http:// + upstream名称

```bash
location / { 
      proxy_pass http://demo; 
}
```

### Nginx配置https支持

1. 去申请https证书，某云有免费版

   <img src="/Users/caining/Library/Application Support/typora-user-images/image-20210719082231549.png" alt="image-20210719082231549" style="zoom:50%;" />

2. 设置证书绑定域名

   ![image-20210719082415629](/Users/caining/Library/Application Support/typora-user-images/image-20210719082415629.png)

3. 配置指纹信息

<img src="/Users/caining/Library/Application Support/typora-user-images/image-20210719083439526.png" alt="image-20210719083439526" style="zoom:50%;" />

1. 下载ssl 证书

   <img src="/Users/caining/Library/Application Support/typora-user-images/image-20210719082546858.png" alt="image-20210719082546858" style="zoom:50%;" />

4. Nginx部署下载的test.pem&test.key两个文件，到此我们绑定的域名https://demo.com，便反向代理到了http://demo.com

```bash
 
 
 
 ##
        # add cnn SSL Settings
        ##
        server{
                listen 443;
                server_name demo.com;
                ssl on;
                ssl_certificate /etc/nginx/cert/test.pem;
                ssl_certificate_key /etc/nginx/cert/test.key;
                ssl_session_timeout 5m;
                location / {
                							#根域名或者ip
                                proxy_pass http://demo.com;
                        }
        }
```





