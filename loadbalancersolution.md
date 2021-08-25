#### Load Balancer Solution With Apache

The users will access the application by using a single public IP address/DNS name of the Load Balancer instead of using separate IP addresses/ DNS names of each Web Server. This ensures a single point of entry to the application through the Load Balancer and also helps to create a better user experience by distributing the traffic evenly between the Web Servers. 

#### Task: To configure an Apache Load Balancer using an Ubuntu EC2 machine to serve the Tooling Website.
#### Prerequisites:
1. Two RHEL8 Web Servers
2. One MySQL DB Server (based on Ubuntu 20.04)
3. One RHEL8 NFS server

#### Configuring an Apache Load Balancer

#### Step 1: Create an Ubuntu Server 20.04 EC2 instance and name it as "apache-lb" 

On the Apache Load Balancer EC2 instance, open TCP traffic inbound on Port 80. Also allow all traffic from load balancer to the web servers on their security group.

 #### Step 2: Install Apache 2 Load Balancer on the EC2 instance
 
 ```
 # Install Apache2
 sudo apt update
 sudo apt install apache2 -y
 sudo apt-get install libxml2-dev
 ```
 ```
 #Enable following modules:
 sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
```

- Ensure that Apache2 is up and running:

![project 8b](https://user-images.githubusercontent.com/41236641/130787868-6089a42a-3780-4fcf-8e45-f596db24741a.JPG)

#### Step 3: Configure Load Balancer:

```
sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://172.31.40.174:80 loadfactor=5 timeout=1
               BalancerMember http://172.31.46.8:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
![project 8](https://user-images.githubusercontent.com/41236641/130787192-7c7def33-9cae-460e-851d-8c7703f3323f.JPG)

#Restart apache server

sudo systemctl restart apache2
```
##### Verify the configuration to see if the Load Balancer is serving as the entry point for allowing traffic into and out of the two web servers. Type the Public IP address or DNS name of the load balancer in the browser as follows:
```
http://3.137.186.32/index.php
```
The tooling website is now accessible through the Load Balancer
![image](https://user-images.githubusercontent.com/41236641/130788594-15ae8c83-f943-4187-9015-fdd499b530bb.png)

For project 7, I mounted /var/log/httpd/ from the Web Servers to the NFS server. I unmounted them and created a separate directory for each web server.
Then, I accessed the logs on each web server:

```
sudo tail -f /var/log/httpd/access_log
```

I also noticed how the logs kept adding new entries every time I refreshed the http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php on the web browser. The traffic was distributed between the two web servers. 
 
##### Local DNS Names Resolution:
 
 - It can get tedious to maintain a long list of IP addresses for the web servers. We can configure local domain name resolution by adding entries to the /etc/hosts file. We can configure IP address to domain name mapping for the Load Balancer.
 
 ```
 sudo vi /etc/hosts
 ```
 Add the following entries:
 ```
 172.31.40.174 Web1
 172.31.46.8 Web2
 ```
  
  Update the configuration in the Load Balancer configuration file:
  ```
  sudo nano /etc/apache2/sites-available/000-default.conf
  ```
  Modify the IP address with the local domain names as follows:
  ```
  BalancerMember http://Web1:80 loadfactor=5 timeout=1
  BalancerMember http://Web2:80 loadfactor=5 timeout=1
  ```
  ![Project 8a](https://user-images.githubusercontent.com/41236641/130789139-0a4b391c-1643-4d41-8f5f-abd90366167b.JPG)

- Upon using the curl command, the web server is accessible on the load balancer:
  ```
  curl http://Web1
  ```
  
  ###### This is only internal configuration and it is also local to the Load Balancer server.
  ###### These names will neither be ‘resolvable’ from other servers internally nor from the Internet.
  
