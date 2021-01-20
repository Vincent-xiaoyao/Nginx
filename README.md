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

