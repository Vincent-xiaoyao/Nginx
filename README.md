<h1>Nginx启用Let’s Encrypt SSL证书</h1>

<p>Let’ s Encrypt 是一个免费的 SSL/TLS 证书发行机构, 证书有效期为90天, 到期前30内可续期，因此不需要担心费用问题。</p>
<h3>服务器环境：</h3>
<hr>
  <ul>
    <li>nginx-1.10.1</li>
    <li>php-7.0.4</li>
    <li>mariadb-10.1.13</li>
  </ul>
<p>用证书的主要过程包括：客户端安装、获取证书、配置Nginx、证书自动续期等几个方面。</p>
<h3>客户端下载</h3>
<hr>
<p>Let’ s Encrypt客户端现已更名为certbot，客户端的地址为：<a href=https://github.com/certbot/certbot/releases>https://github.com/certbot/certbot/releases </a></p>
<pre>
  <div>#下载
  wget  https://github.com/certbot/certbot/archive/v0.22.2.tar.gz</div>
  <div>#解压
  tar xzvf v0.22.2.tar.gz</div>
  <div>#进入目录
  cd certbot-0.22.2</div>
</pre>
运行一次客户端，进行检查升级：
<h3>获取证书</h3>
<hr>
<p>申请过程中要验证绑定的域名是否属于申请人, 其原理就是申请人在域名所在的服务器上申请证书, 然后 Let’ s Encrypt 会访问绑定的域名与客户端通信成功即可通过。</p>
<p>验证的方式有两种，一种是停止当前的 web server 服务, 让出 80 端口, 由客户端内置的 web server 启动与 Let’ s Encrypt 通信；一种是在域名根目录下创建一个临时目录,
  并要保证外网通用域名可以访问这个目录，这种方式不需要停止当前的 web server 服务。</p>
<p>证书获取方式1：通过访问80端口方式验证</p>
<pre>
  <div>#停止nginx
  systemctl stop nginx</div>
  <div>#获取证书, --standalone 参数:使用内置web server. --email 参数:管理员邮箱,证书到期前会发邮件到此邮箱提醒. -d 参数:要绑定的域名,同一域的不同子域都要输入.
  ./certbot-auto certonly --standalone --email admin@4spaces.org -d 4spaces.org -d www.4spaces.org</div>
  <div>#启动nginx
  systemctl start nginx</div>
</pre>
<p>证书获取方式2：通过临时目录验证</p>
<pre>
  <div>#--webroot 参数:指定使用临时目录的方式. -w 参数:指定后面-d 域名所在的根目录, 如果一次申请多个域的, 可以附加更多 -w...-d... 这段.
  ./certbot-auto certonly --webroot --email admin@4spaces.org -w /usr/share/nginx/html -d 4spaces.org -d www.4spaces.org</div>
</pre>
完成上面的操作即可获得 SSL 证书, 保存在 “/etc/letsencrypt/live/根域名/” 目录下, 会产生 4 个文件, 其中3个证书文件, 1个私钥文件. 不要移动证书的位置, 以免续期时出现错误。
关于Letsencrypt使用的更多命令参见<a href="https://www.4spaces.org/certbot-command-line-tool-usage-document/">「这里」</a>。

<h1>配置Nginx启用https</h1>

<p>上面你的Nginx配置并没有启用ssl，下面我们需要开始配置nginx，让其支持https。进行这一步的前提是你前面已经成功生成证书。</p>
<p>编辑文件/etc/nginx/conf.d/default.conf(我是通过yum的方式安装的nginx，配置目录在这里，你根据自己的情况来)，进行如下配置（这个是我的完整配置）：</p>
<pre>
  #设置非安全连接永久跳转到安全连接
server{
    listen 80;
    server_name 4spaces.org www.4spaces.org;
    #告诉浏览器有效期内只准用 https 访问
    add_header Strict-Transport-Security max-age=15768000;
    #永久重定向到 https 站点
    return 301 https://$server_name$request_uri;
}
server {
   #启用 https, 使用 http/2 协议, nginx 1.9.11 启用 http/2 会有bug, 已在 1.9.12 版本中修复.
   listen 443 ssl http2;
   server_name 4spaces.org www.4spaces.org;
   #首页
   index  index.php index.html index.htm;
   #网站根目录
   root   /usr/share/nginx/4spaces;
   #告诉浏览器当前页面禁止被frame
   add_header X-Frame-Options DENY;
   #告诉浏览器不要猜测mime类型
   add_header X-Content-Type-Options nosniff;

    #证书路径
    ssl_certificate /etc/letsencrypt/live/4spaces.org/fullchain.pem;
    #私钥路径
    ssl_certificate_key /etc/letsencrypt/live/4spaces.org/privkey.pem;
    #安全链接可选的加密协议
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    #可选的加密算法,顺序很重要,越靠前的优先级越高.
    ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    #在 SSLv3 或 TLSv1 握手过程一般使用客户端的首选算法,如果启用下面的配置,则会使用服务器端的首选算法.
    ssl_prefer_server_ciphers on;
    #储存SSL会话的缓存类型和大小
    ssl_session_cache shared:SSL:10m;
    #缓存有效期
    ssl_session_timeout 60m;

    location / {
        try_files $uri $uri/ /index.php?$args;  #修改内容
    }

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #修改此处内容支持php
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root/$fastcgi_script_name;
        include        fastcgi_params;
    }

}
</pre>
<h3>证书续期</h3>
<hr>
<p>前面说了，证书的有效期是3个月，你可以在证书过期前的30天内，进行续期，也可以进行脚本自动续期。</p>
方式1
进入你在下载的certbot客户端目录，执行证书续期的脚本命令如下：
