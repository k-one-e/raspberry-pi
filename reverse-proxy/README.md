# Nginx as Reverse Proxy on Raspberry Pi

Instruction how to expose your internal applications securely to the internet with Nginx as reverse proxy

- Update repositories and install prerequisites
>sudo apt-get update
>sudo apt-get -y install build-essential libpcre3 libpcre3-dev openssl libssl-dev zlib1g zlib1g-dev
- Download latest stable version of Nginx and extract it
	- wget https://nginx.org/download/nginx-1.14.2.tar.gz
	- tar -xzf nginx-1.14.2.tar.gz
	- cd nginx-1.14.2/
- Add ssl module before compiling
	- ./configure --with-http_ssl_module
- Compile and install
	- make
	- sudo make install
- Create a symbolic link to nginx
	- sudo ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/nginx

- You should now be able to start Nginx and request the default page using curl on 127.0.0.1
	- sudo nginx 
	- curl 127.0.0.1
- Stop the nginx instance with
	- sudo nginx -s stop

- To make nginx auto start on every boot
	- git clone https://github.com/Fleshgrinder/nginx-sysvinit-script.git
	- cd nginx-sysvinit-script
	- sudo make

#Configuration
- For Basic HTTP Auth 
	- sudo apt-get -y install apache2-utils
	- sudo htpasswd -c /usr/local/nginx/conf/.htpasswd user@mail.com

- Backup the default Nginx Config file and recreate a new config file
	- mv /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf
	- nano /usr/local/nginx/conf/nginx.conf


- To make any application on your local network securely accessible through the reverse proxy create a .conf file for each application inside conf/sites-enabled/ 
For example the conf file to access your internal grafana instance through the reverse proxy would be like the following:

grafana.conf:
server {
    listen 443 ssl;
    server_name grafana.*;
    location / {
      proxy_pass http://IP-To-Grafana-Instance:3000;
    }
}

