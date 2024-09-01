1) Create a VPC called `task-vpc` with a IPv4 CIDR `192.168.0.0/16`
2) Create 4 subnets: 2 public and 2 private with CIDRs `192.168.x.0/24` where x = 1, ..., 4
3) Create an Internet Gateway called `igw-task` and attach it to `vpc-task`
4) Create a Route Table called `pb-rtb` with a route `0.0.0.0/0` to forward requests to the `igw-task`
5) Edit the route tables of `pb-subnet-1` and `pb-subnet-2` to be `pb-rtb`
6) Create an EC2 instance `proxy-lb`:
- Key pair: `my-rsa-key`
- Subnet: `pb-subnet-1`
- Auto-assign public IP: Enable
- Security group:
	- Name: `pb-subnet-1-secgrp`
	- Description: public subnet 1 security group for (proxy-lb) loadbalancer and (proxy1) ec2 instance.
	- Rules: add `HTTP` with `Anywhere` source type
7) Create an EC2 instance `proxy1` in `pb-subnet-1`
8) Create an EC2 instance `proxy2` in `pb-subnet-2` with `pb-ubnet-2-secgrp` security group with an HTTP group rule.
---
9) Connect to `proxy-lb`
- `sudo yum install nginx -y`
- `cd /etc/nginx/conf.d` and create a new conf file `lb.conf`
- paste and change the private IPs of `proxy1` and `proxy2`:
```
upstream backend_servers {
    server proxy1.pv.ipv4;
    server proxy2.pv.ipv4;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---
10) Connect to `proxy1`:
- `sudo yum install nginx -y`
- `cd /usr/share/nginx/html`
- Modify the `index.tml` file to show PROXY 1 in the welcome page

---
11) Connect to `proxy1`:
- `sudo yum install nginx -y`
- `cd /usr/share/nginx/html`
- Modify the `index.tml` file to show PROXY 2 in the welcome page

---
12) Check your work by accessing the public IP of `proxy-lb`

---
13) Create an EC2 instance called `app-lb`:
- Subnet: `pv-subnet-1`
- Auto-assign public IP: Disable   # (Private subnet)
- Security group:
	- Name: `pv-subnet-1-secgrp`
	- Description: private subnet 1 security group for (app-lb) loadbalancer and (app1) ec2 instance.
	- Rules: add HTTP with port `80`and `Custom` source type which is 192.168.0.0/16 (Desc.: allow only for vpc-task)

---
14) Create an EC2 instance called `app1`:
- Subnet: `pv-subnet-1`
- Security group : `pv-subnet-1-secgrp`

---
15) Create an EC2 instance called `app2`:
- Subnet: `pv-subnet-2`
- Security group : `pv-subnet-1-secgrp`

---
16) Connect to `proxy1` and `proxy2`:
- `cd /etc/nginx/conf.d`
- `sudo vim proxy.conf`
- paste and change the IP to be the private IP of `app-lb`
```
server {
    listen 80;

    location / {
        proxy_pass http://app-lb.pv.ipv4;
    }
}
```

---
17) Create a NAT Gateway called `natgw-task` at subnet `pb-subnet-1` then update the route table of the private subnets to have a `0.0.0.0/0` route to a NAT Gateway `natgw-task`

---
18) Since we can't access the private instances directly, we will use `proxy2` to ssh into `app-lb`, `app1` and `app2`
- Copy your key contents to `proxy2`

- ssh into `app-lb` using the key:
- `sudo su`
- `yum install nginx -y`
- `cd /etc/nginx/conf.d`
- Create a `lb.conf` file with backend servers that are private IPs of `app1` and `app2`

- ssh into `app1` and `app2` using the key:
- `sudo su`
- `yum install nginx -y`
- `cd /usr/share/nginx/html`
- Modify `index.html` file to view app1's IP or app2's IP in the welcome page

---
