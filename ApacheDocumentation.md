
# Apache Manual installation on Ubuntu 16.04

1. Download Apache Http server from [Apache website](http://httpd.apache.org/download.cgi) (e.g httpd-2.2.13.tar.gz)
2. Unzip it using tar command  ```  tar xvfz httpd-2.2.13.tar.gz ```
3. Install Dependencies
    ##### APR          
    ```
    cd /home/username/hms/installs/apache/4.0/apr 
    wget https://www-us.apache.org/dist//apr/apr-1.7.0.tar.gz
    tar -xvzf apr-1.7.0.tar.gz
    cd apr-1.7.0/
    ./configure
    make    
    make install
    ```
    ##### APR Utils
    ```
    cd  /home/username/hms/installs/apache/4.0/apr-util
    wget https://www-us.apache.org/dist//apr/apr-util-1.6.1.tar.gz
    tar -xvzf apr-util-1.6.1.tar.gz
    cd apr-util-1.6.1/
    ./configure --with-apr=/home/username/hms/installs/apache/apr
    make
    make install
    ```
    ##### pcre
    ```
    cd  /home/username/hms/installs/apache/4.0/pcre
    wget https://ftp.pcre.org/pub/pcre/pcre2-10.33.tar.gz
    tar -xvzf pcre2-10.33.tar.gz
    cd pcre2-10.33/
    ./configure
    make 
    make install
    ```
4. Configure Apache
     ``` ./configure --prefix=/home/username/hms/installs/apache/4.0 –enable-shared=max --with-apr=/home/username/hms/installs/apache/4.0/apr/ -with-pcre=/home/username/hms/installs/apache/4.0/pcre```
 in prefix you enter your installation directory. 
 6. If it configured successfully build it with ``` make ```command. 
 7. Install with ```sudo make install``` command. 
 8. To start the Apache Server just go to the your installation path 
(installs/apache/bin) and run command ```sudo apachectl -k start```

### Basic commands in Apache
* To **start** go to (apache/bin) and type `apachectl -k start`
* To **stop** go to (apache/bin) and type `apachectl -k stop`
* To **restart** go to (apache/bin) and type `apachectl -k restart`
* To **restart gracefully** go to (apache/bin) and type `apachectl -k graceful`
* To **stop gracefully** go to (apache/bin) and type `apachectl -k graceful-stop`
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
    
***  
 # Creating SSL Certificate for Apache
 Here we're going to create a self signed certificate for Apache server
 1. Create the SSL certificate using following code. 
``` 
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```
we are creating the certificate and key using the OpenSSL. In `-keyout` we set path for creating our key and `out` we mention the path for certificate. This command will ask simple questions like country name, state, email then you fill them accordingly. 
For OpenSSL we should create a strong Diffie-Hellman group. That can be done by 
`sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048` command. 

 2. Create Apache configuration snippet.   
 For that go to (apache/conf/extra) and open *Httpd-ssl.conf* file and add below commands. Before you do that get a ***backup*** for that file. 

```
SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
SSLProtocol All -SSLv2 -SSLv3
SSLHonorCipherOrder On
Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains"
Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff
# Requires Apache >= 2.4
SSLCompression off 
SSLSessionTickets Off
SSLUseStapling on 
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"

SSLOpenSSLConfCmd DHParameters "/etc/ssl/certs/dhparam.pem"
```
 3. Change the below codes in the same file. 
    ```
    #For server name you enter your's accordingly 
    ServerName example.com
    #give the location of your certificate you created earlier
    SSLCertificateFile      /etc/ssl/certs/apache-selfsigned.crt
    #location for key file created earlier
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
    ```
4. Enable modules in Apache.
For that go to (apache/conf) and open httpd.conf and uncomment below lines. 
    ```
    LoadModule ssl_module modules/mod_ssl.so
    LoadModule headers_module modules/mod_headers.so
    LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
    Include conf/extra/httpd-ssl.conf
    ```
5. Now restart the Apache Server
    `apachectl -k restart`
6. When you go to https://example.com it will show an error in red saying self signed certificates can not be trusted. And if you click the *Not secure* icon you can see the certificate created by you. 
 
 
    









