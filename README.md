# LoadBalancerSolutionWithApache

As a continuation of project "DevOps Tooling Solution", we have this project. 

Keeping all the web server, nfs and database configurations the same, we will now go ahead and configure the load balancer.

1) Launch an EC2 instance on AWS with following specification:
      * t2 micro
      * Ubuntu
      * Number of instances -1 
      * Name: LoadBlancer
      
2) Add a security group to allow all the connections on TCP (Select http protocol and 0.0.0.0/0).

3) Open the terminal and SSH into the load balancer.

4) Now, we start the load balancer initial set-up using the commands below:
     
          ``````````
          #install apache
          sudo apt update -y
          sudo apt install apache2 -y
          sudo apt-get install libexml2-dev -y
          
          sudo a2enmod rewrite
          sudo a2enmod proxy
          sudo a2enmod proxy_balancer
          sudo a2enmod proxy_http
          sudo a2enmod headers
          sudo a2enmod lbmethod_bytraffic
          
          sudo systemctl restart apache2
          sudo systemctl status apache2
          ````````
 5) We are now configuring the load balancer using the following commands:
          
          ````
          sudo vi /etc/apache2/sites-available/000-default.conf
          
          #Add this configuration into this section <VirtualHost *:80>  </VirtualHost>
          
          <Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
          </Proxy>

          ProxyPreserveHost On
          ProxyPass / balancer://mycluster/
          ProxyPassReverse / balancer://mycluster/
          
          #Restart apache server

          sudo systemctl restart apache2
          ````
          
       Side Note:
       Here we are using bytraffic method which distributs load between web servers according to current traffic load. We can control in which proportion the traffic can be distributed by loadFactor parameter. Other methods are bybusiness, byrequest and heartbeat
       
   6) Restart apache2 and verify if we can access webpage via the loadbalancer.
   
   7) Next, if in previous project, we have mounted /var/log/httpd from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.
   Open two ssh/Putty consoles for both Web Servers and run following command:
   
          ````
          sudo tail -f /var/log/httpd/access_log
          ````
          
   8) Now try and refresh both webservers 1 and 2.
   
   9) Go to sudo vi /etc/hosts file to configure the DNS. Map the DNS names to the private IP addresses. Next go to sudo vi /etc/apache2/sites-available/000-default.conf
Change the Private IP address previously configured to DNS names.

  10) Now check if the DNS resolution works correctly by accessing the webpage via curl http://Web1
  
  
  -------------------------------------------------------------------------------
          
