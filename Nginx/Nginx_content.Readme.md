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
