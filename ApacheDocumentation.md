
# Apache Manual installation on Ubuntu 16.04

 1. Download Apache Http server from [Apache website](http://httpd.apache.org/download.cgi) (e.g httpd-2.2.13.tar.gz)
 2. Unzip it using tar command  ```  tar xvfz httpd-2.2.13.tar.gz ```
 3. Configure folder location
    ``` ./configure --prefix=/usr/local/apache –enable-shared=max ```
in prefix you enter your installation directory (/home/username/hms/installs/apache)
 4. If you got error saying “*APR not found*” then run the below command.
    ```sudo apt-get install libapr1-dev libaprutil1-dev```    
 5. Again if you are come up with “*pcre-config for libpcre not found*” error run,
 ```sudo apt-get install libpcre3-dev```
 6. If it configured successfully build it with ``` make ```command. 
 7. Install with ```sudo make install``` command. 
 8. To start the Apache Server just go to the your installation path 
(installs/apache/bin) and run command ```sudo apachectl -k start```
 ***

# Reverse Proxy Configuration

1.  Go to your apache configuration file (installs/apache/conf) and open the ***httpd.conf*** file. 
2.  In there uncomment the below lines,
    ``` 
    Include etc/extra/httpd-vhosts.conf 
    LoadModule proxy_module modules/mod_proxy.so
    LoadModule proxy_http_module modules/mod_proxy_http.so
    ```
3.  Then go to (apache/conf/extra/) and open ***httpd-vhosts.conf*** file and add below lines.
    ```
    <VirtualHost *:80>
        ServerName example.com
        ServerAlias www.example.com
        ServerAdmin webmaster@example.com

        ProxyRequests Off
        <Proxy *>
          Order deny,allow
          Allow from all
        </Proxy>
        
        ProxyPass / http://127.0.0.1:8080/webModule/
        ProxyPassReverse / http://127.0.0.1:8080/webModule/

        <Location />
          Order allow,deny
          Allow from all
        </Location>

    </VirtualHost>
    ```
4.  Then go to (/etc) and open hosts file and add **example**.**com** in front of 127.0.0.1 
5.  Restart the apache server by  ```sudo apachectl -k restart``` 
6.  In browser when you type **example**.**com** you will be redirect to `http://127.0.0.1:8080/webModule/`

***
# Apache Load Balancer Configuration

1. First of all you need to create a multiple instances of tomcat server. 
2. For that copy (/opt/tomcat) folder and create (/opt/tomcat2).
3. Go to tomcat2/conf/ and open server.xml file and change the port numbers accordingly.
4. Go to (/etc/systemd/system) and copy ***tomcat.service*** and create ***tomcat2.service***.
5. In tomcat2.service, change
    CATALINA_PID=/opt/tomcat2/temp/tomcat.pid 
    CATALINA_BASE=/opt/tomcat2
6. Then start both tomcat instances using below commands. 
    ```
    systemctl start tomcat.service
    systemctl start tomcat2.service
    ```
    So now tomcat will start on localhost:8080 port and tomcat2 will start on localhost:8090 port as configured on *server.xml* file.
7. Go to apache configuration folder (apache/conf) and open *httpd.conf* file. and uncomment below lines.
    ```
    LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
    LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
    LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
    ```
8. Then go to (apache/conf/extra/) and open *httpd-vhosts.conf* file and make changes as below. 
    ```
    <VirtualHost *:80>
	    ServerName example.com
        ServerAlias www.example.com
        ServerAdmin webmaster@example.com

    <Proxy balancer://mycluster>        
        BalancerMember http://127.0.0.1:8080/webModule/
	    BalancerMember http://127.0.0.1:8090/webModule/
	    ProxySet lbmethod=byrequests	
    </Proxy> 
    ProxyRequests Off
    <Proxy *>
        Order deny,allow
        Allow from all
    </Proxy>
    ProxyPass /balancer-manager ! 
    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
    
    <Location /balancer-manager>
   	    SetHandler balancer-manager
    </Location>
   </VirtualHost>
    ```
    in here *mycluster* is our balancer name and we have 2 Balancer Members we assigned for earlier made 2 tomcat servers. 
    The method we here using for load balancing is **byrequsts**. There are other methos also like as **bytraffic, bybusyness** and **byheatbeat**
    From going to the link/balancer-manager (example.com/balancer-manager)
    you can see the stats. 

    ![](https://i.postimg.cc/2SGFR9XP/Screenshot-from-2019-09-05-12-56-41.png)
    
    









