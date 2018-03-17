# nginx,uwsgi,django实现负载均衡服务器




### 手动安装Python环境



​	sudo apt-get update
​	sudo apt-get install software-properties-common
​	sudo add-apt-repository ppa:jonathonf/python-3.6
​	sudo apt-get update
​	sudo apt-get install python3.6
​	cd /usr/bin
​	ls | grep python
​	sudo rm python
​	sudo ln -s python3.6m python
​	sudo apt-get install python3-pip
​	pip --version
​	sudo python pip install --upgrade pip
​	pip --version	





### 安装nginx服务器



####安装zlib依赖库

​	sudo apt-get install zlib1g-dev

####解压相关文件（点此下载）

​	tar -xzvf nginx-1.11.3.tar.gz

​	tar -xzvf openssl-1.0.1.tar.gz

​	tar -xzvf pcre-8.41.tar.gz

####进入Nginx解压目录

​	cd /nginx-1.11.3/         #根据实际路径
####配置环境

	 ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre=../pcre-8.41 --with-openssl=../openssl-1.0.1
####编译

	make
####注意：
如果出现“pcre.h No such file or directoryy"则安装(无错误请跳过)
​	
	sudo apt-get install libpcre3 libpcre3-dev

####安装

	sudo make install
####说明
	nginx会被安装在/usr/local/nginx目录下

####相关命令
	启动nginx服务：

		sudo /usr/local/nginx/sbin/nginx
	关闭Nginx服务：
	
		sudo /usr/local/nginx/sbin/nginx -s stop
	重新加载配置
	
		sudo /usr/local/nginx/sbin/nginx -s reload
	指定配置文件
	
		sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
	查看版本信息
		
		sudo /usr/local/nginx/sbin/nginx -v
		sudo /usr/local/nginx/sbin/nginx -V
	查看80端口被占用情况
		netstat -ano | grep 80
	关闭占用80端口的程序 sudo fuser -k 80/tcp


####测试
	启动服务

		sudo /usr/local/nginx/sbin/nginx

	打开浏览器，输入服务器ip地址
	如果是阿里云，需在安全组开发80端口
###安装django

	sudo pip install Django==1.11.4
创建django项目关闭调试允许任何机器访问
DEBUG=False
ALLOW_HOSTS=['*',]

###安装uwsgi

	pip install uwsgi

在django工程目录下创建名为uwsgi.ini的文件
​	
	vim uwsgi.ini
写入以下内容
	[uwsgi]
	#使用nginx链接时使用
	socket =127.0.0.1:8000
	#直接做web服务器使用
	#http=127.0.0.1:8000  #如果不用ngix可以打开这个，关闭socket
	#项目目录
	chdir=/home/...#你的项目工程目录
	wsgi-file=project/wsgi.py
	processes=4
	threads=2
	master=True
	pidfile=uwsgi.pid #进程文件名
	daemonize=uwsgi.log



启动

	uwsgi --ini uwsgi.ini

停止
	uwsgi--stop uwsgi.pid

​	
####配置nginx
打开nginx配置文件
​	
	vim /usr/local/nginx/conf/nginx.conf

配置如下
	sendfile on
	top_nopush on;
	tcp_nodelay on; #需要自行添加
	gzip on;
	
	#实现负载均衡
	
	upstream mycon{
	  server 47.93.253.250:8000；#此处端口应当与你nwsgi的端口一样
	  server 47.93.253.251:8000
	}


	#server中 
		server_name: localhost;
		charset utf-8
		location /{
	      
	      	includ uwsgi_params;
	      	uwsgi_pass 127.0.0.1:8000;
	      	}
	      location /static {
	        alias /var/www/a/static/;#目录自己定义
	      }
		}


####重新加载配置文件

		sudo /usr/local/nginx/sbin/nginx -s reload
####c创建静态文件目录

	sudo mkdir -vp /var/www/a/static/#与nginx中设置的的目录相同
	sudo chmod 777 /var/www/a/static/
####迁移静态文件
	进入工程项目
	vim settings.py
	更改如下
	STATIC_ROOT='/var/www/a/static/
	迁移静态文件
	python manage.py collectstatic

