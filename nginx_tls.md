## Load Balancer Solution With Nginx and SSL/TLS

It's critical we secure our client-server communcation with a more secured protocol such as HTTPS. This is to prevent security threats one of which is called Man-In-The-Middle (MIMT) attack.

SSL and its newer version, TSL - is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server..

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

### Task

In this project we'll register our website using LetsEnrcypt Certificate Authority, to automate certificate issuance we'll use a shell client recommended by LetsEncrypt - cetrbot.

- Configure Nginx as a Load Balancer
- Register a new domain name and configure secured connection using SSL/TLS certificates

Your target architecture will look like this:
*image archi

## Configure Nginx As A Load Balancer

You can either uninstall Apache from the existing Load Balancer server, or create a fresh installation of Linux for Nginx (I prefer the latter).

1. Create a VM based on Ubuntu Server 20.04 LTS and name it Nginx LB.
> Don't forget to open TCP port 80 and 443, for HTTP and secured HTTPS connections respectively)

2. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

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