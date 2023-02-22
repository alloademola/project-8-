# project-8-
Load Balancer Solution With Apache

After we complete Project-7 you might wonder how a will a user access each of the webservers
using 3 different IP addreses or 3 different DNS names. 
we might also wonder what is the point of having 3 different servers doing exactly the same thing.

When we access a website in the Internet we use an URL and 
we do not really know how many servers are out there serving our requests,
this complexity is hidden from a regular user, but in case of websites that are being visited 
by millions of users per day (like Google or Reddit) it is impossible t
o serve all the users from a single Web Server (it is also applicable to databases, 
but for now we will not focus on distributed DBs).

Each URL contains a domain name part, which is translated (resolved) to IP address of a target server 
that will serve requests when open a website in the Internet. 
Translation (resolution) of domain names is perormed by DNS servers,
the most commonly used one has a public IP address 8.8.8.8 and belongs to Google. 
we can try to query it with nslookup command:

nslookup 8.8.8.8
Server:  UnKnown
Address:  103.86.99.99

Name:    dns.google
Address:  8.8.8.8

When we have just one Web server and load increases – we want to serve more and more customers, 
we can add more CPU and RAM or completely replace the server with a more powerful one – this is called "vertical scaling". 
This approach has limitations – 
at some point we may reach the maximum capacity of CPU and RAM that can be installed into your server.

Another approach used to cater for increased traffic is "horizontal scaling" – distributing load across multiple Web servers. 
This approach is much more common and can be applied almost seamlessly and almost infinitely 
(we can imagine how many server Google has to serve billions of search requests).

Horizontal scaling allows to adapt to current load by adding (scale out) or removing (scale in) Web servers. 
Adjustment of number of servers can be done manually or automatically (for example, based on some monitored metrics like CPU and Memory load).

Property of a system (in our case it is Web tier) to be able to handle growing load by adding resources, is called "Scalability".

In our set up in Project-7 we had 3 Web Servers and each of them had its own public IP address and public DNS name. A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of even 3 server, let alone millions of Google servers.

In order to hide all this complexity and to have a single point of access with a single public IP address/name, a Load Balancer can be used. 
A Load Balancer (LB) distributes clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

Let us take a look at the updated solution architecture with an LB added on top of Web Servers 
(for simplicity let us assume it is a software L7 Application LB, for example – Apache, NGINX or HAProxy)
  
  As seen in the below 
  
  Fig 1
  
  <img width="795" alt="Screenshot 2023-02-22 at 05 58 28" src="https://user-images.githubusercontent.com/118350020/220526323-e1c2d5d5-a4db-4826-87a8-9a7974e0c5d8.png">

In this project we will enhance our Tooling Website solution by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.

Task

Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 intance. 
Make sure that users can be served by Web servers through the Load Balancer.

To simplify, let us implement this solution with 2 Web Servers, 
the approach will be the same for 3 and more Web Servers.

Prerequisites


1: Two RHEL8 Web Servers
2: One MySQL DB Server (based on Ubuntu 20.04)
3: One RHEL8 NFS server

Fig 2

<img width="722" alt="Screenshot 2023-02-22 at 06 03 33" src="https://user-images.githubusercontent.com/118350020/220527027-cc443ade-4f08-45b0-8ae3-9bdb1e78faec.png">

Now let us 

CONFIGURE APACHE AS A LOAD BALANCER

so we are going to 
Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb
As seen the in  

Fig 3 below

<img width="1440" alt="Screenshot 2023-02-22 at 07 06 15" src="https://user-images.githubusercontent.com/118350020/220537140-b39b3226-1987-4a07-851c-0ec70e142556.png">

Now let us connect from the terminal as seen in

Fig 4 below

<img width="1029" alt="Screenshot 2023-02-22 at 07 10 15" src="https://user-images.githubusercontent.com/118350020/220537742-b47589b2-0dc2-421a-9265-621dd9537eff.png">

Next thing to do now is to Open TCP port 80 on Project-8-apache-lb 
by creating an Inbound Rule in Security Group.
As we can see in 

Fig 5 below


<img width="1327" alt="Screenshot 2023-02-22 at 07 17 26" src="https://user-images.githubusercontent.com/118350020/220538909-192f64f1-4a16-4acc-b1ad-5c052e643647.png">

next thing now is to install Apache2 on our load balancer terminal, 
but we have to first update our terminal, So let us run the command below, one after each other.

sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

Now let us
Enable the following modules: Running the below command at once

sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

As shown in 

Fig 6 below

<img width="1093" alt="Screenshot 2023-02-22 at 07 49 46" src="https://user-images.githubusercontent.com/118350020/220545021-9b78b17a-6251-4efc-a3be-33cff4a70a9c.png">

next is to Restart apache2 service using the below command

sudo systemctl restart apache2

Let us check the status of our apache2 to see if it is active by running the command below

sudo systemctl status apache2
As you can see from 

Fig 7 below

<img width="1019" alt="Screenshot 2023-02-22 at 07 57 28" src="https://user-images.githubusercontent.com/118350020/220546207-e5f0b245-f30a-46c3-b81a-95c3a09118e7.png">

Next step is to configure load balancing by running the below command

sudo vi /etc/apache2/sites-available/000-default.conf

As shown in

Fig 8 below


<img width="1025" alt="Screenshot 2023-02-22 at 08 01 00" src="https://user-images.githubusercontent.com/118350020/220547243-73670aad-bb7f-4215-81b5-1b263f86f541.png">

So next step is to  

Add this configuration into this section <VirtualHost *:80>  </VirtualHost> on Fig 8 above

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

As shown in the below 

Fig 9        


<img width="1025" alt="Screenshot 2023-02-22 at 08 20 48" src="https://user-images.githubusercontent.com/118350020/220550785-654114e8-0954-4cab-9501-54d13af067db.png">


Now let us Restart our apache server using the below command 


sudo systemctl restart apache2

as shown on Fig 10 below

<img width="1023" alt="Screenshot 2023-02-22 at 08 24 56" src="https://user-images.githubusercontent.com/118350020/220551562-dad6bb72-597c-40c5-b4d2-fd864f15a4e3.png">

Now let us Verify that our configuration works – 
try to access your LB’s public IP address or Public DNS name from your browser:
 
 As shown on fig 11, 12 and 13  below
 
 
 <img width="1440" alt="Screenshot 2023-02-22 at 08 28 36" src="https://user-images.githubusercontent.com/118350020/220552313-956bc718-9134-425b-8e13-4411b061da72.png">

Fig 12

<img width="1440" alt="Screenshot 2023-02-22 at 08 31 21" src="https://user-images.githubusercontent.com/118350020/220552828-c03ee9dd-67fe-4745-a477-987a9b26a770.png">


 fig 13
 
 
 <img width="1440" alt="Screenshot 2023-02-22 at 08 31 34" src="https://user-images.githubusercontent.com/118350020/220552901-d7c2fa77-da2f-474c-b453-a83f5a9883f7.png">


Note: If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.

So here is the command to get that done

sudo umount -f /var/log/httpd/

let us try to see the accesss log of our webserver1 by running the below command
sudo tail -f /var/log/httpd/access_log

fig 14

<img width="1025" alt="Screenshot 2023-02-22 at 08 57 50" src="https://user-images.githubusercontent.com/118350020/220563364-aad676a2-a541-47ca-8524-cbccf5a26c4f.png">


the highlighted part on fig 14 above ,  
are the server request from those who tried to access 
these our webserver 

if we refresh our browser page http://18.197.128.112/admin_tooling.php
several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. 
The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers 
it means that traffic will be disctributed evenly between them.

If we configured everything correctly our users will not even notice 
that their requests are served by more than one server.
As shown in 

Fig 15 below


<img width="1026" alt="Screenshot 2023-02-22 at 09 33 43" src="https://user-images.githubusercontent.com/118350020/220565888-f2d685de-8b56-4cc7-8750-534cff69ba44.png">



Optional Step – Configure Local DNS Names Resolution

Sometimes it is tedious to remember and switch between IP addresses, 
especially if we have a lot of servers under our management.
What we can do, is to configure local domain name resolution. 

So the easiest way is to use /etc/hosts file, although this approach is not very scalable,
but it is very easy to configure and shows the concept well. 
So let us configure IP address to domain name mapping for our LB.

So let us Open this file below on our LB server by running the command below and see fig 16 below as well

sudo vi /etc/hosts


fig 16

<img width="1029" alt="Screenshot 2023-02-22 at 09 48 36" src="https://user-images.githubusercontent.com/118350020/220569480-67d7ceae-416e-4e2f-83e5-771d698725be.png">

so let us  Add 2 records into this file above with Local IP address and arbitrary name for both of our Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2

172.31.6.9 web1
172.31.8.66 web2

as shown in 

Fig 17 below

<img width="1041" alt="Screenshot 2023-02-22 at 09 56 41" src="https://user-images.githubusercontent.com/118350020/220571143-51bc613b-37d2-4a92-ba68-fa5ef5390cd0.png">

Now let us update our LB config file with those names instead of IP addresses.

we are going to run the below command to get that done as shown on Fig 18 below

sudo vi /etc/apache2/sites-available/000-default.conf

Fig 18


<img width="1028" alt="Screenshot 2023-02-22 at 10 04 10" src="https://user-images.githubusercontent.com/118350020/220572789-7c992ecf-dfd5-435a-a60f-ab82271543f8.png">

Fig. 19


<img width="1035" alt="Screenshot 2023-02-22 at 10 07 11" src="https://user-images.githubusercontent.com/118350020/220573756-b68aefaf-31d6-41ea-9203-dc73ef144f3b.png">


so let us try to curl our Web Servers from LB locally using the below command as shown on fig 20

curl http://Web1 

fig 20 


<img width="1023" alt="Screenshot 2023-02-22 at 10 14 25" src="https://user-images.githubusercontent.com/118350020/220575455-9a5c65fe-df29-43dd-b4ec-c95652d5f59d.png">
<img width="1027" alt="Screenshot 2023-02-22 at 10 14 53" src="https://user-images.githubusercontent.com/118350020/220575524-ec1dd76a-2375-4052-b538-786afa7f87a6.png">

As you can see from the diagram above, the local name resolution work perfectly

 now let  uus try to curl web2 using the below command. as shown on fig 21 below
 
 curl http://Web2
 
 Fig. 21
 
 
 <img width="1024" alt="Screenshot 2023-02-22 at 10 20 57" src="https://user-images.githubusercontent.com/118350020/220577048-46675c85-aa7f-4965-973d-a40895e85206.png">
<img width="1026" alt="Screenshot 2023-02-22 at 10 21 21" src="https://user-images.githubusercontent.com/118350020/220577070-cebab502-0bd5-4dd5-a33e-c4244a7ca5e0.png">

as you can see from the above diagram, it works.

So it means that, all our configuration worked out.


NOte this is only internal configuration and it is also local to your LB server, 
these names will neither be ‘resolvable’ from other servers internally nor from the Internet.

Targed Architecture
Now your set up looks like the diagram below


<img width="635" alt="Screenshot 2023-02-22 at 10 27 54" src="https://user-images.githubusercontent.com/118350020/220579041-9b002f26-ffc8-43e8-b149-de332f71e55f.png">

<img width="702" alt="Screenshot 2023-02-22 at 10 28 48" src="https://user-images.githubusercontent.com/118350020/220579203-8b9eaeda-0252-4ed6-89e4-012f9729bf4f.png">

Congratulations!
You have just implemented a Load balancing Web Solution for your DevOps team.
Project 8 ending.
