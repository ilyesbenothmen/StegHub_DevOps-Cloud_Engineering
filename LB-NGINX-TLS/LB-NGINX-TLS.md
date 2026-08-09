##  Load Balancer Solution With Nginx and SSL/TLS
### Introduction:


This project consist of two parts:
1. Configure Nginx as a load balancer
2. Register a new (hostname) subdomain on a free dynamic DNS service. and configure secured connection using SSL/TLS certificates.

The target architecture is illustrated in the diagram below.

![alt](images/architecture.png)

### Part1 Configure Nginx As A Load Balancer:


In this part , we will install and configure Nginx on ubuntu

![alt](images/1.png)

Download the certificate, then establish a terminal connection to the server using its public IP address.

![alt](images/2.png)

Ensure that the newly created EC2 instance is launched in the same Availability Zone and within the same VPC subnet (172.31.16.0/20) as the two web servers deployed in the previous lab.

In addition, configure inbound security group rules to allow HTTP and HTTPS traffic on ports 80 and 443.

>[!NOTE]
>These two considerations help prevent connectivity and communication issues.

![alt](images/3.png)

Add the IP addresses of web1 and web2 to /etc/hosts to improve clarity and ease of use.

![alt](images/4.png)

As recommended, refresh the system packages to resolve dependencies and ensure access to the latest package versions.

```bash
sudo apt update
```

![alt](images/5.png)

Install nginx with
```bash
sudo apt install nginx
```

![alt](images/6.png)

Configure /etc/nginx/nginx.conf to include web1 and web2 as the upstream and comment out the line that include sites-enabled directory.

![alt](images/7.png)

Restart Nginx and verify that the service is running correctly.
```bash
sudo systemctl restart nginx
sudo systemctl status nginx
```


![alt](images/8.png)

At this point the Nginx load balancer is up and running !

### Part2 Register a new domain name and configure secured connection using SSL/TLS certificates:

At this stage, the Tooling website is accessible, but we must ensure that access is secure and reliable (for example, protected against man‑in‑the‑middle attacks). 
To achieve this, we rely on SSL/TLS, which is designed to secure web communications.  
  
The prerequisites are:  
1. A public hostname that can be resolved via public DNS.  
2. A valid certificate issued by a trusted Certification Authority (CA) recognized by modern web browsers.

To obtain a public service name, you can either:
1. Purchase a domain from a well-known registrar such as GoDaddy, Domain.com, or Bluehost.
2. Use a free subdomain provided by a popular dynamic DNS (DDNS) service.

In this lab, we selected DuckDNS as our free DDNS provider and reserved the hostname toolings.duckdns.org.

![alt](images/9.png)

After each EC2 instance reboot, the public IP address may change. As a result, we must update the IP–hostname association to ensure that www.toolings.duckdns.org continues to resolve correctly.

![alt](images/9-1.png)

To address this issue permanently, allocate an Elastic IP address to the EC2 instance. An Elastic IP is a static, public IPv4 address provided by AWS that persists across instance stops, starts, and reboots.

![alt](images/10.png)

Associate the allocated Elastic IP with the EC2 instance running NGINX.

![alt](images/11.png)

Verify in the EC2 instances inventory that the Elastic IP is correctly associated with the target EC2 instance.

![alt](images/12.png)


Open `/etc/nginx/nginx.conf`, update the `domain.com` entries to `www.toolings.duckdns.org`, then restart the NGINX service to apply the changes.

![alt](images/15.png)

Update DuckDNS with the newly allocated Elastic IP by setting the current IP for your hostname to the EIP address.

![alt](images/13.png)


Next, we’ll install Certbot and request an SSL/TLS certificate.

First, verify that the `snapd` service is up and running:

```bash
sudo systemctl status snapd
```

If it’s not active, start and enable it:

```bash
sudo systemctl enable --now snapd
```

![alt](images/16.png)

Install Certbot using Snap:

```bash
sudo snap install --classic certbot
```

![alt](images/17.png)
Create a symbolic link so `certbot` is available system-wide:

```bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

![alt](images/18.png)
Request an SSL/TLS certificate with:

```bash
sudo certbot --nginx
```

When prompted, select the desired domain (option 1 in this case).

![alt](images/19.png)

Test secure access by visiting `https://www.toolings.duckdns.org` in your browser. You should see a padlock icon and a message indicating that the connection is secure.

![alt](images/19-1.png)

Click the padlock icon in the browser’s address bar to inspect the SSL/TLS certificate details.


Inspect the certificate details such as:

- **Common Name (CN)**
- **Organization**
- **Valid To (expiration date)**

In this case, the certificate is visibly issued by **Let’s Encrypt** and is valid until **November 4, 2026**.

![alt](images/20.png)
Now, try navigating the site by first performing authentication.
![alt](images/21.png)

Verify that the website is functioning as expected .

![alt](images/22.png)

By default ,LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in `dry-run` mode

```bash
sudo certbot renew --dry-run
```


![alt](images/23.png)

Best practice is to have a scheduled job that to run `renew` command periodically. let us configure a `cronjob` to run the command twice a day.
To do so,lets edit the crontab file with the following command

```bash
crontab -e
```
and add the following
```bash
* */12 * * * root /usr/bin/certbot renew > /dev/null 2>&1  
```

![alt](images/24.png)



### Conclusion:
In this project we have just implemented an Nginx load balancing web solution with secured HTTPS connection with periodically updated SSL/TLS certificates.
This choice is well suited for our lab scenario. However, in real-world deployments, requirements differ: for example, public companies may be required by local regulations to obtain certificates from approved national certificate authorities.
