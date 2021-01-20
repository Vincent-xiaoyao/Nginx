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
<pre class="prettyprint"><code class=""><span class="com">#下载</span><span class="pln">
wget  https</span><span class="pun">:</span><span class="com">//github.com/certbot/certbot/archive/v0.22.2.tar.gz</span><span class="pln">

</span><span class="com">#解压</span><span class="pln">
tar xzvf v0</span><span class="pun">.</span><span class="lit">22.2</span><span class="pun">.</span><span class="pln">tar</span><span class="pun">.</span><span class="pln">gz

</span><span class="com">#进入目录</span><span class="pln">
cd certbot</span><span class="pun">-</span><span class="lit">0.22</span><span class="pun">.</span><span class="lit">2</span></code></pre>
