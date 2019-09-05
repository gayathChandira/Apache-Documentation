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
3.  
