# Setting up Auto Scaling

Prior to the auto scaling setup:
- Ensure a working application AMI  is available; a tutorial of the setup can be found in [Creating an AMI](https://github.com/PutuJem/tech230_AWS/blob/main/ec2_mongodb_ami.md).
- The OS AMI should be ubuntu version 18.04 with a t2.micro instance type.
- Create a launch template from the application AMI.

 >***Important note***: the application state may not be running once the template is created; in this case, enter the provision script below into the template user data.

### **User Data Launch Template script**

```bash
#!/bin/bash

cd ~/

# Update and upgrade the package manager.

sudo apt-get update -y

sudo apt-get upgrade -y

# Install the Nginx web server.

sudo apt-get install nginx -y

# Overwrite the contents of the default configuration file and output the new file contents.

sudo sed -i 's+try_files $uri $uri/ =404;+proxy_pass http://localhost:3000;+' /etc/nginx/sites-available/default

# change the environment variable

echo 'export DB_HOST=mongodb://<mongodb_private_IP_address>/posts' >> .bashrc

# Start the Nginx web server; remember to use the `enable` to enable a service on next system restart.

sudo systemctl stop nginx

sudo systemctl start nginx

sudo systemctl enable nginx

# install application dependant Node packages

sudo apt-get install python-software-properties -y

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -

sudo apt-get install nodejs -y

sudo npm install pm2 -g

# Get the app folder from a GitHub repo

git clone https://github.com/PutuJem/tech230_AWS.git ~/app

# Navigate to the app folder

cd ~/app/app/app/

# Stop all running processes, in case there are any, then run the application

pm2 stop all

npm install

node seeds/seed/js

pm2 start app.js --update-env
```



1. Navigate to the "Auto Scaling Groups" and "Create an Auto Scaling Group".

![](autoscaling_setup_images/1_create.png)

2. Create a name and select an appropriate launch template.

![](autoscaling_setup_images/2_name.png)

3. Select the default VPC and availability zones for where the instances will be deployed; availability zones enable scalability and high availability, in the occurence of one zone being down the other zones can still be accessed.

![](autoscaling_setup_images/3_network.png)

4. Attach a new load balancer, as shown in the example below, and select "Application Load Balancer" with an appropriate name. Ensure to also amend the scheme to "Internet-facing", as the application is being accessed through the internet.

 > Note: Turn on the Elastic Load Balancing health checks, as recommended by AWS.

![](autoscaling_setup_images/4_load.png)

5. Under "Listeners and routing, create a new target group and name it appropriately.

![](autoscaling_setup_images/5_listen.png)

6. Set a suitable group size.

![](autoscaling_setup_images/6_group.png)

7. Select the "Target tracking scaling policy" and set the rules to scale out as intended.

![](autoscaling_setup_images/7_policy.png)

8. Create a tag to assist in identifying the auto scaling group; in example, set the key as "Name" and value as "tech230-james-ASG".

9. Review the configuration of the scaling group and when ready, create the auto scaling group.

10. To test the application is working as intended, navigate to the "Load balancer" tab within EC2 services and select the recently created auto scaling group.

11. Locate the DNS address and enter it into the web address, the application should be displayed as shown. 

![](autoscaling_setup_images/8_balancer.png)

![](autoscaling_setup_images/9_dns.PNG)

### **Automating the MongoDB setup**

```bash
#!/bin/bash

cd ~/

sudo apt-get update -y

sudo apt-get upgrade -y

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927

echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

sudo apt-get update -y

sudo apt-get upgrade -y

sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20

sudo sed -i 's+bindIp: 127.0.0.1+bindIp: 0.0.0.0+' /etc/mongod.conf

sudo systemctl restart mongod

sudo systemctl enable mongod
```