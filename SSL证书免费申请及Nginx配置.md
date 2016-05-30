#SSL证书免费申请及Nginx配置
##Let's Encrypt 证书申请

###官方网站
https://letsencrypt.org/getting-started/
###下载安装 certbot
可以参照网站https://certbot.eff.org/

	
	git clone https://github.com/certbot/certbot
	cd certbot
	./certbot-auto certonly --webroot -w /opt/www/  -d xxx.com -d www.xxx.com
	./certbot-auto certonly --standalone -d xxx.com -d www.xxx.com  #该行代码可以不用执行，直接执行下边一行代码
	./certbot-auto renew --dry-run
	./certbot-auto renew
	
	执行完后如果没有错误，所有生成文件会在/etc/letsencrypt/目录下
	具体网站的证书在/etc/letsencrypt/live/目录下
	


注意事项
	
	1.python需要升级到2.7或以上版本
	2.新安装的python2.7需要执行easy_install virtualenvwrapper
	3.执行 standalone 或者renew --dry-run命令时，需要停止当前的80端口监听，即停止apache或者nginx服务，执行完后再启动
	
打开/etc/letsencrypt/live/目录文件说明

cert.pem #server cert only  
privkey.pem #private key  
chain.pem #intermediates  
fullchain.pem #server cert + intermediates  

###Nginx配置
可以打开https://mozilla.github.io/server-side-tls/ssl-config-generator/参考配置

dhparam.pem文件生成：

	openssl dhparam -out dhparam.pem 2048
	
ssl_trusted_certificate 文件生成
	
	cd /etc/letsencrypt/live/xxx.com
	wget https://letsencrypt.org/certs/isrgrootx1.pem
	mv isrgrootx1.pem root.pem
	cat root.pem chain.pem > root_ca_cert_plus_intermediates

Nginx配置文件


	listen 443;
	ssl on;
    ssl_certificate /etc/letsencrypt/live/xxx.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/xxx.com/privkey.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    ssl_dhparam /etc/nginx/conf.d/dhparam.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
    ssl_prefer_server_ciphers on;


    add_header Strict-Transport-Security max-age=15768000;

    ssl_stapling on;
    ssl_stapling_verify on;

    ssl_trusted_certificate /etc/letsencrypt/live/xxx.com/root_ca_cert_plus_intermediates;
    resolver dns17.hichina.com dns18.hichina.com;
    
  resolver 后面，你需要找到域名服务器的提供商，填写域名解析服务器地址
  Nginx 配置完成后，重启后，用浏览器测试是否一切正常。

###测试SSL服务是否正常
打开网址
https://www.ssllabs.com/ssltest/index.html
输入域名，等待检验

遇到问题
	
	1.静态文件，如图片,js无法访问，或者报错，可以在nginx 静态文件节点添加
	add_header Content-Security-Policy upgrade-insecure-requests;
	2.引用第三方http请求报js console错误，确定第三方网站支持https,将http改为https

###自动化定期更新证书
Let's Encrypt 建议每隔60天更新一次证书
执行命令：

	./certbot-auto renew
	nginx -s reload
