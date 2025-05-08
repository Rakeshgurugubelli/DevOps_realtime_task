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

chown -R www-data:www-data

.conf files will be paced on /etc/nginx/sites-available/

Once we place configuration files in sites-available we need to enabled it then only it will work 

***Realtime projects******************************

******hosting static website*********

code: https://github.com/jaiswaladi246/static-site.git

Step-1: Copy the code to var/www/

![image](https://github.com/user-attachments/assets/57a123ee-c97e-4229-bec4-d116786e59d8)

Step-2: change the permissions

sudo chown -R www-data:www-data static-site/

![image](https://github.com/user-attachments/assets/efa73f55-3474-4cc2-ba63-a5e0c76646f3)

Step-3: create the configuration file 

cd /etc/nginx/sites-available/

![image](https://github.com/user-attachments/assets/32ebf565-063e-4fdf-bbbf-e4b3478e4486)

Step-4: Enable Site and reload the nginx 

sudo ln -s /etc/nginx/sites-available/dev.conf /etc/nginx/sites-enabled/

sudo nginx -t

sudo systemctl reload nginx

**dev.conf**

    server {
        listen 80;
# in server name we will place domain name or dns or ip
        server_name 3.6.37.254;
        location / {
# webpage code placed in the below path
            root /var/www/static-site;
# homepage
           index index.html;
            try_files $uri $uri/ =404;
        }
    }
![image](https://github.com/user-attachments/assets/b0a3957c-bc47-41f9-a17a-28650642f04f)


******Nginx webserver a reverse proxy*********

In reverse proxy it will hide the access to the backend server when front end wants to access to backend,
it should be using nginx frst then the nginx will sends request to backend from backend nginx will take request
and nginx will send request to frontend

code:https://github.com/jaiswaladi246/nginx-node-proxy.git

Step-1: Copy the Frontend code to /var/www/

![image](https://github.com/user-attachments/assets/e8259e9f-47fa-4cbc-bd60-f180f920e034)

Step-2: change the permissions

sudo chown -R www-data:www-data frontend/

![image](https://github.com/user-attachments/assets/c5e0851f-b5a0-4ba8-b5aa-82be99e256e7)

Step-3: create the configuration file 

cd /etc/nginx/sites-available/

**for frontend**

![image](https://github.com/user-attachments/assets/06583158-936c-449e-9602-abca88292368)

**for backend**
-----------------------**config file**---------------------------------------------------------------------------
server {
        listen 80;
# in server name we will place domain name or dns or ip
        server_name 3.111.170.61;

        location / {
# webpage code placed in the below path

            root /var/www/frontend;
# homepage
            index index.html;
            try_files $uri $uri/ =404;
        }

# if we type domainname/api then proxy pass to api 
location /api/ {
# local host change to ip address

        proxy_pass http://3.111.170.61:3000;  # Backend service

# it is basically makesure connection is alive
        proxy_http_version 1.1;
# $host:it basically refers to host it means domain name
        proxy_set_header Host $host;
# $remote_addr: it basically refers to client ip address
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

       # websocket support
       location /socket.io/ {
# local host change to ip address

        proxy_pass http://3.111.170.61:3000;  # Backend service

# it is basically makesure connection is alive
        proxy_http_version 1.1;
                  


 # Optional: Handle WebSocket connections
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }

}
-----------------------------------------------------------------------------------------------------------
To run backend based on our code need to install node.js on machine
and also we are not using package.json to install packages so we need to install manually
---***node js**-----

# Download and install nvm:

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# in lieu of restarting the shell

\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:

nvm install 22

# Verify the Node.js version:

node -v # Should print "v22.15.0".
nvm current # Should print "v22.15.0".

# Verify npm version:

npm -v # Should print "10.9.2".

---package install--

npm init -y
npm install express.js

to start backend -- node index.js

Step-4: Enable Site and reload the nginx 

sudo ln -s /etc/nginx/sites-available/frontend.conf /etc/nginx/sites-enabled/

# if it fails it means config file syntax is incorrect
sudo nginx -t

sudo systemctl reload nginx

![image](https://github.com/user-attachments/assets/f3f49c1a-b20e-4298-bee6-235e998394e5)

once we hit the frontend automatically backend will receive it and we can see logs 

*****************Nginx loadbalancer*****************************

code: https://github.com/jaiswaladi246/nginx-loadbalancer.git

Step-1: Copy the Frontend code to /var/www/

![image](https://github.com/user-attachments/assets/e8259e9f-47fa-4cbc-bd60-f180f920e034)

Step-2: change the permissions

sudo chown -R www-data:www-data frontend/

![image](https://github.com/user-attachments/assets/c5e0851f-b5a0-4ba8-b5aa-82be99e256e7)

Step-3: create the configuration file 

cd /etc/nginx/sites-available/

**loadbalancer.conf**

upstream backend_servers {
        # List of backend app servers
        server 13.233.92.115:3001;
        server 13.233.92.115:3002;
    }

    server {
        listen 80;
# in server name we will place domain name or dns or ip
        server_name 3.111.170.61;

        location / {
# webpage code placed in the below path

            root /var/www/frontend;
# homepage
            index index.html;
            try_files $uri $uri/ =404;
        }

# if we type domainname/api then proxy pass to api 
location /api/ {
# local host change to ip address

        proxy_pass http://3.111.170.61:3000;  # Backend service

# it is basically makesure connection is alive
        proxy_http_version 1.1;
# $host:it basically refers to host it means domain name
        proxy_set_header Host $host;
# $remote_addr: it basically refers to client ip address
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

       # websocket support
       location /socket.io/ {
# local host change to ip address

        proxy_pass http://3.111.170.61:3000;  # Backend service

# it is basically makesure connection is alive
        proxy_http_version 1.1;
                  


 # Optional: Handle WebSocket connections
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }

}

Step-4: Enable Site and reload the nginx 

sudo ln -s /etc/nginx/sites-available/frontend.conf /etc/nginx/sites-enabled/

# if it fails it means config file syntax is incorrect
sudo nginx -t

sudo systemctl reload nginx

once we access with frontend automatically request goes to backend 1 & backend 2 based on load




