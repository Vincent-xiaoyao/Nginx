<h1>Nginx启用Let’s Encrypt SSL证书</h1>

<p>Let’ s Encrypt 是一个免费的 SSL/TLS 证书发行机构, 证书有效期为90天, 到期前30内可续期，因此不需要担心费用问题。</p>
<h3>服务器环境：</h3>
<hr>
  <ul>
    <li>nginx-1.10.1</li>
    <li>php-7.0.4</li>
    <li>mariadb-10.1.13</li>
  </ul>
<hr>
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
<h3>获取证书</h3><hr>
<p>申请过程中要验证绑定的域名是否属于申请人, 其原理就是申请人在域名所在的服务器上申请证书, 然后 Let’ s Encrypt 会访问绑定的域名与客户端通信成功即可通过。</p>
<p>验证的方式有两种，一种是停止当前的 web server 服务, 让出 80 端口, 由客户端内置的 web server 启动与 Let’ s Encrypt 通信；一种是在域名根目录下创建一个临时目录, 并要保证外网通用域名可以访问这个目录，这种方式不需要停止当前的 web server 服务。</p>
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







