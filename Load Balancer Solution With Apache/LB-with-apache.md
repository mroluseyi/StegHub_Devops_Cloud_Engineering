# Load Balancer Solution With Apache

This documentation will guide you through the process of how i was able to setup an Apache load balancer on a new EC2 instance running Ubuntu 24.04 to distribute traffic between my two web servers. In the previous documentation , i already configured the 3 tier architecture with 2 webservers.

## Task

Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 instance. Make sure that users can be served by Web servers through the Load Balancer.

## Prerequisites

Ensure that the following servers are installedd and configure already.

- Two RHEL9 Web Servers

- One MySQL DB Server (based on Ubuntu 24.04)

- One RHEL9 NFS Server

## Prerequisites Configurations

- Apache (httpd) is up and running on both Web Servers.

- /var/www directories of both Web Servers are mounted to /mnt/apps of the NFS Server.

- All neccessary TCP/UDP ports are opened on Web, DB and NFS Servers.

- Client browsers can access both Web Servers by their Public IP addresses or Public DNS names and can open the Tooling Website (e.g, http://<Public-IP-Address-or-Public-DNS-Name>/index.php)

## Step 1 - Configure Apache As A Load Balancer

## 1. Create an Ubuntu Server 24.04 EC2 instance and name it Project-8-apache-lb

![alt text](Images/ec2-lb-details.PNG)

## 2. Open TCP port 80 on Project-8-apache-lb by creating an Inbounb Rule in Security Group

![alt text](Images/lb-security-rule.PNG)

## 3. Install Apache Load Balancer on Project-8-apache-lb and configure it to point traffic coming to LB to both Web Servers.

#### i. Install Apache2

- Access the instance

`ssh -i /path/to/your-key.pem ubuntu@your-load-balancer-public-ip`

- Update and upgrade Ubuntu

`sudo apt update && sudo apt upgrade -y`

- Install Apache

`sudo apt install apache2 -y`

![alt text](Images/apache-installed.PNG)

`sudo apt-get install libxml2-dev`

![alt text](Images/dependencies-installed.PNG)

#### ii. Enable the following modules

```
sudo a2enmod rewrite

sudo a2enmod  proxy

sudo a2enmod  proxy_balancer

sudo a2enmod  proxy_http

sudo a2enmod  headers

sudo a2enmod  lbmethod_bytraffic
```

![alt text](Images/modules-enabled.PNG)

#### iii. Restart Apache2 Service

```
sudo systemctl restart apache2
sudo systemctl status apache2
```

![alt text](Images/apache-status-check.PNG)

### Configure Load Balancing

#### i. Open the file 000-default.conf in sites-available directory.

`sudo vi /etc/apache2/sites-available/000-default.conf`

#### ii. Add this configuration into the section <VirtualHost \*:80> </VirtualHost>

```
<Proxy “balancer://mycluster”>
            BalancerMember http://172.31.14.213:80 loadfactor=5 timeout=1
           BalancerMember http://172.31.4.112:80 loadfactor=5 timeout=1
           ProxySet lbmethod=bytraffic
           # ProxySet lbmethod=byrequests
</Proxy>


ProxyPreserveHost on
ProxyPass / balancer://mycluster/
ProxyPassReverse / balancer://mycluster/
```

![alt text](Images/apache-server-config.PNG)

#### iii. Restart Apache

`sudo systemctl restart apache2`

`bytraffic` balancing method with distribute incoming load between the Web Servers according to currentraffic load. The proportion in which traffic must be distributed can be controlled bt loadfactor parameter.

Other methods such as `bybusyness`, `byrequests`, `heartbeat` can also be adopted.

## 4. Verify the configuration works

#### i. Access the website using the LB's Public IP address or the Public DNS name from a browser

![alt text](Images/lb-website-accessed.PNG)
Note: If in the previous project, /var/log/httpd was mounted from the Web Server to the NFS Server, unmount them and ensure that each Web Servers has its own log directory.

#### ii. Unmount the NFS directory

- Check if the Web Server's log directory is mounted to NSF

```
df -h
sudo umount -f /var/log/httpd
```

If the directory is busy, the services using it needs to be stopped first.

`sudo systemctl stop httpd`

- Check that the directory is unmounted

` df -h`

#### iii. Open two ssh consoles for both Web Server and run the command:

`sudo tail -f /var/log/httpd/access_log`

#### iv. Refresh the browser page several times and ensure both Web Servers receive HTTP and GET requests. New records must apear in each web server log files. The number of request to each servers will be approximately the same since loadfactor is set to the same value for both servers. This means that traffic will be evenly distributed between them.

Web server 1 `access_log`
![alt text](Images/server1-access-logs.PNG)

Web server 2 `access_log`

![alt text](Images/server2-access-logs.PNG)

## Optional Step - Configure Local DNS Names Resolution

Sometimes it is tedious to remember and switch between IP addresses, especially if there are lots of servers to manage. It is best to configure local domain name resolution. The easiest way is use `/etc/hosts` file, although this approach is not very scalable, but it is very easy to configure and shows the concept well.

### Configure the IP address to domain name mapping for our Load Balancer.

#### Open the hosts file

`sudo vi /etc/hosts`

#### Add two records into file with Local IP address and arbitrary name for the Web Servers

![alt text](Images/dns-hosts.PNG)

#### Update the LB config file with those arbitrary names instead of IP addresses

`sudo vi /etc/apache2/sites-available/000-default.conf`

```
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```

![alt text](Images/dns-apache-config.PNG)

#### Try to curl the Web Servers from LB locally

`curl http://Web1`

![alt text](Images/curl-web1.PNG)

`curl http://Web2`

![alt text](Images/curl-web2.PNG)
