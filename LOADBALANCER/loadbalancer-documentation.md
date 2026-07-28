##  Load Balancer Solution With Apache
### Introduction:
When users access a website or web application, they typically interact with a single domain or subdomain and receive a seamless response. From the user's perspective, the application appears to be hosted on a single server. In reality, however, the service is often delivered by a pool of backend servers working together to ensure high availability, scalability, and fault tolerance. The component responsible for distributing incoming requests across these backend servers is the load balancer.

A load balancer is either a dedicated hardware appliance or a software solution that distributes client requests among multiple backend services. These services may consist of web servers, application servers, database servers, or any other type of workload requiring traffic distribution. By preventing any single server from becoming overloaded, a load balancer improves application performance, resilience, and overall service availability.

There are two common approaches to scaling an infrastructure:

Vertical scaling (Scaling Up): Increasing the capacity of an existing server by adding resources such as CPU cores, memory, or storage.
Horizontal scaling (Scaling Out): Increasing the application's capacity by deploying additional server instances and distributing the workload among them.

A load balancer plays a key role in horizontal scaling by efficiently routing requests according to a configurable balancing strategy. Common algorithms include Round Robin, Least Connections, Weighted Distribution, and Traffic-Based balancing. The choice of algorithm depends on the application's requirements, server capabilities, and expected workload.

In this lab, we will implement a software load balancer using Apache HTTP Server (mod_proxy_balancer). The load balancer will be deployed on an Ubuntu EC2 instance hosted in AWS and configured to distribute HTTP requests across two backend web servers located within the same Virtual Private Cloud (VPC). This architecture demonstrates a typical enterprise deployment pattern that improves both availability and scalability while maintaining a single entry point for client requests.s.



![alt](images/diagram.png)

### Step0 preparing prerequisites:


![alt](images/0.png)

![alt](images/2.png)


![alt](images/2-1.png)


### Step1 install Apache and Update the Firewall

>[!TIP]
>

![alt](images/3.png)


![alt](images/4.png)


![alt](images/5.png)

![alt](images/6.png)

![alt](images/7.png)
> [!NOTE]
>
![alt](images/9.png)

### Step2 Installing Mysql

![alt](images/10.png)



![alt](images/11.png)

![alt](images/13.png)



### Step3 Installing PHP



![alt](images/15.png)


>[!Note]
>To host multiple services on the same server, we need to configure virtual hosts (vhosts).

### Step4 Creating a virtual host for your website using Apache


![alt](images/19.png)




![alt](images/16.png)



![alt](images/20.png)


![alt](images/17.png)

### Step 5 Enable PHP on the website:
viour , you'll need to edit /etc/apache2/mods-enabled/dir.conf 
![alt](images/21.png)


![alt](images/18.png)

### Conclusion:

