**Nginx is a webserver**

**Ex**: when we want to visit a website we will place a url in the browser it will forwad the request to the webserver

**Longback a ago(old process before webserver)**
we are having 2 types of webservers 

  . **process based webserver:** 
  
    Ex: Let's take 100 persons hits the url when users placed a url in the browser 100 process(process will have own memory) will be created
        if load increases within few minutes server will be overloaded and it will crash

  . **Thread based webserver:**  

    Ex: Let's take 10 persons hit's the url process will create thread for Each request by using this method processor will save memory
        if load increases within few minutes server will be overloaded and it will crash

**To overcome above problem  we are using Nginx (or)  Engine-X**

-> it is solving c10k problem (it means more users hits the applications chances of crashing website)

-> Nginx is a event driven(when request comes processor will create ) and Asynchronous webserver(don't sit idle do something)

-> if we created virtual machine having 4 core and installed nginx it means 4 processor will be created

**Workflow of nginx**

start nginx

TCP connection(3 way hand shake: when user sends the request it will go to servers using syn(synchronous) and sever sent back  sync+acknowledge to browser)

. nginx can also perform reverse proxy server

. nginx can also use as load balancer

. nginx can also be used as http cache

. nginx can hold static website

. nginx can also be used as api gateway

**Security groups open on server**

**Inbound rule**

port range: 443,80,22

**outbount rule**

port range: All traffic

**Installation steps of nginx on ubuntu server**

Step-1: install

Sudo apt update && apt-get install -y nginx

Step-2: Reload & check status

systemctl restart nginx (or) nginx -s reload

systemctl status nginx 

**files existing in /etc/nginx/**

webpages need to place on /var/www/ and ensure user is www_data

chown -r www_data:www_data

.conf files will be paced on /etc/nginx/sites-available/

Once we place configuration files in sites-available we need to enabled it then only it will work 

***Realtime projects******************************

* hosting static website

  

