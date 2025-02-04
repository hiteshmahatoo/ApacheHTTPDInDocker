# ApacheHTTPDInDocker
Apache HTTPD in Docker with Loadbalancing using HTTP & AJP Protocols

1. First Create a Dockerfile with only the below line
FROM httpd:2.4

2. Create a image with the Dockerfile
docker build -t apache2image .

3. Run the container with the image
docker run -dit --name http-official-container -p 8080:80 apache2image

4. Copy the httpd.conf file from the container to the local.
docker cp http-official-container:/usr/local/apache2/conf/httpd.conf .

5. Now in Dockerfile Add the below line according to your Dockerfile workspace path
COPY httpd.conf /usr/local/apache2/conf/httpd.conf

Now you are ready to customize the httpd.conf and create a image to run the container 


To connect to any one application server below are the steps:

1. Uncomment the lines in httpd.conf
   LoadModule proxy_module modules/mod_proxy.so
   LoadModule proxy_http_module modules/mod_proxy_http.so

2. Then enter the below lines in the last of httpd.conf
   <VirtualHost *:80>
         ServerName localhost

         ProxyPreserveHost On
         ProxyPass / http://192.168.51.48:8081/
         ProxyPassReverse / http://192.168.51.48:8081/

         ErrorLog /usr/local/apache2/logs/error.log
         CustomLog /usr/local/apache2/logs/access.log combined
   </VirtualHost>

Save it then recreate the docker image and run the container the configs will reflect.
================================================

To connect to two or more applications servers with http protocol below are the steps:

1. Uncomment the lines in httpd.conf
   LoadModule proxy_module modules/mod_proxy.so
   LoadModule proxy_http_module modules/mod_proxy_http.so
   LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
   LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
   LoadModule proxy_balancer_module modules/mod_proxy_balancer.so

2. Add the below lines at the end of httpd.conf
   <VirtualHost *:80>
   ProxyPreserveHost On
   <Proxy "balancer://mycluster">
      Order deny,allow
      Allow from all
      BalancerMember "http://192.168.51.48:8081" route=node1
      BalancerMember "http://192.168.51.48:8082" route=node2
      ProxySet lbmethod=byrequests
      ProxySet stickysession=JSESSIONID
   </Proxy>
   ProxyPass        "/" "balancer://mycluster/"
   ProxyPassReverse "/" "balancer://mycluster/"
   </VirtualHost>

Save it then recreate the docker image and run the container the configs will reflect.

To connect to two or more applications servers with AJP protocol below are the steps:

1. Uncomment the lines in httpd.conf
   LoadModule proxy_module modules/mod_proxy.so
   LoadModule proxy_http_module modules/mod_proxy_http.so
   LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
   LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
   LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
   LoadModule proxy_ajp_module modules/mod_proxy_ajp.so

2. Add the below lines at the end of httpd.conf
   <VirtualHost *:80>
   ProxyPreserveHost On
   <Proxy "balancer://mycluster">
        Order deny,allow
        Allow from all
        BalancerMember "ajp://192.168.51.48:8009" route=node1 secret=local
        BalancerMember "ajp://192.168.51.48:8010" route=node2 secret=local
        ProxySet lbmethod=byrequests
        ProxySet stickysession=JSESSIONID
   </Proxy>
   ProxyPass        "/" "balancer://mycluster/"
   ProxyPassReverse "/" "balancer://mycluster/"
   </VirtualHost>

Save it then recreate the docker image and run the container the configs will reflect.
