## Load Balancer Solution With Nginx and SSL/TLS

It's critical we secure our client-server communcation with a more secured protocol such as HTTPS. This is to prevent security threats one of which is called Man-In-The-Middle (MIMT) attack.

SSL and its newer version, TSL - is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server..

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

### Task

In this project we'll register our website using LetsEnrcypt Certificate Authority, to automate certificate issuance we'll use a shell client recommended by LetsEncrypt - cetrbot.

- Configure Nginx as a Load Balancer
- Register a new domain name and configure secured connection using SSL/TLS certificates

Your target architecture will look like this:

![](https://github.com/Arafly/Nginx_LB_TLS/blob/master/assets/nginx_lb.png)

## Configure Nginx As A Load Balancer

You can either uninstall Apache from the existing Load Balancer server, or create a fresh installation of Linux for Nginx (I prefer the latter).

1. Create a VM based on Ubuntu Server 20.04 LTS and name it Nginx LB.
> Don't forget to open TCP port 80 and 443, for HTTP and secured HTTPS connections respectively).

2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

```
#Open this file on your LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

`sudo apt update && sudo apt install nginx`

Ensure nginx is running and enable it across reboots

`sudo systemctl start nginx && sudo systemctl enable nginx`

```
Synchronizing state of nginx.service with SysV service script with /lib/systemd/s
ystemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable nginx
```

`sudo systemctl status nginx`

```
 nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: >
     Active: active (running) since Tue 2021-05-11 21:15:12 UTC; 1min 6s ago
       Docs: man:nginx(8)
   Main PID: 1972 (nginx)
      Tasks: 3 (limit: 4713)
     Memory: 6.1M
     CGroup: /system.slice/nginx.service
             ├─1972 nginx: master process /usr/sbin/nginx -g daemon on; master_p>
             ├─1973 nginx: worker process
             └─1974 nginx: worker process
May 11 21:15:12 nginx-lb systemd[1]: Starting A high performance web server and >
May 11 21:15:12 nginx-lb systemd[1]: Started A high performance web server...
```

4. Create a config file for nginx, and paste in the following lines:

`sudo vi /etc/nginx/sites-available/load_balancer.conf`

```
#insert following configuration into http section

upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }
server {
    listen 80;
    server_name molokofi.ml www.molokofi.ml;
    location / {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```

- Remove the default nginx config file;

`sudo rm -rf /etc/nginx/sites-enabled/default`

- Symbolically link the sites-avalaible and sites-enabled together

`sudo ln -s /etc/nginx/sites-available /etc/nginx/sites-enabled`

> **Gotcha!** You need to understand that the target of a symlink is a pathname. And it can be absolute or relative to the directory which contains the symlink.

Assuming you have load_balancer.conf in sites-available. You'd try:

```
cd sites-enabled
sudo ln -s ../sites-available/load_balancer.conf .

ls -l
Now you will have a symlink in sites-enabled called load_balancer.conf which has a target ../sites-available/load_balancer.conf
```

```
/etc/nginx/sites-enabled$ ll
total 8
drwxr-xr-x 2 root root 4096 May 12 14:03 ./
drwxr-xr-x 8 root root 4096 May 11 21:15 ../
lrwxrwxrwx 1 root root   37 May 12 14:03 load_balancer.conf -> ../sites-available/lo
ad_balancer.conf
```

- Verify the syntax of your Nginx configuration by running:

`sudo nginx -t`

```
Output:

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

- Now reload Nginx (due to the major changes you've made to the config file)

`sudo systemctl reload nginx`

