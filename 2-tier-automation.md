
# Automating a two-tier architecture application

This is a comprehensive guide to setting up a two-tier architecture application containing Sparta Globals web-application and a MongoDB database.

To setup the two-tier architecture, it is recommended that the steps are followed in sequence.

### **Automating the MongoDB instance and configuration files**

Create a new EC2 instance with the suitable configuration for (***in example***):
- OS AMI image (***ubuntu 18.04, normal image***)
- Instance type (***t2.micro***)
- Key pair
- Security Group (***Allowing ports 22, 80, 443, 3000***)
  
Add the script below to the `User Data` field, within the `Advanced` settings.

```bash
#!/bin/bash

cd /home/ubuntu/

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

To test, `ssh` into the instance through git bash and check that mongodb is running and active.

```bash
sudo systemctl status mongod
```

Before proceeding to the next step, ensure the instance is still running and copy its private IPv4 address.

### **Automating the application setup and reverse proxy**

Create a new instance of the application image; ensure use the correct naming convention, key pair `tech230.pem` and security group.

Enter the instance through a new git bash terminal using the ssh command provided in the connect section of the instance summary.

>An example of this is shown below.

```bash
ssh -i "~/.ssh/tech230.pem" root@ec2-3-253-80-245.eu-west-1.compute.amazonaws.com
```

Check the files already present with in the instance.

```bash
ls
```

Create a new provision file to include a script to setup the reverse proxy and start the application.

```bash
touch app-provision.sh
```

Check the user rights and ensure the user is able to execute files.

```bash
ls -l
```

```bash
chmod u+x app-provision.sh
```

> Once the file turns green, the user is able to excute.

![ls page](ls.png)

To setup the automation script, first enter the file.

```bash
sudo nano app-provision.sh
```

Secondly, paste the script below into the `app-provision.sh` file.

> The script provides comments with instructions on what each command accomplishes.

```bash
#!/bin/bash

cd /home/ubuntu/

sudo apt-get update -y

sudo apt-get upgrade -y

sudo apt-get install nginx -y

sudo sed -i 's+try_files $uri $uri/ =404;+proxy_pass http://localhost:3000;+' /etc/nginx/sites-available/default

sudo systemctl restart nginx

sudo systemctl enable nginx

sudo apt-get install python-software-properties -y

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -

sudo apt-get install nodejs -y

sudo npm install pm2 -g

export DB_HOST=mongodb://<db-private-ip>:27017/posts

git clone https://github.com/PutuJem/tech230_AWS.git /home/ubuntu/app

cd /home/ubuntu/app/app/app/

npm install

node seeds/seed.js

pm2 start app.js --update-env

pm2 restart app.js --update-env
```

Save and exit the file using `ctrl+x`.

After starting a new instance, the user will be required to perform a manual command to replace the IP address within the DB_HOST environment variable.

> Amend the `<new_private_IP_address>` as shown in the command below.

```bash
sed -i -e 's/<old_private_IP_address>/<new_private_IP_address>/g' app-provision.sh
```

Navigate to the web browser and enter the public IPv4 address.
