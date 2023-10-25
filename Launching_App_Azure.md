# Deploying App on Azure

### Step 1: Create a VM

Type Virtual Machines in Search Bar and create a Virtual Machine

![Screenshot 2023-10-25 100019.png](images%2FScreenshot%202023-10-25%20100019.png)

Fill in the necessary details, we want to use `Ubuntu 18.04 Pro Gen x2`

![Screenshot 2023-10-25 100421.png](images%2FScreenshot%202023-10-25%20100421.png)

We want to use `Standard_B1s`, and `Use existing key stored in Azure` and select our public key

![Screenshot 2023-10-25 100958.png](images%2FScreenshot%202023-10-25%20100958.png)

Allow HTTP & SSH as this is for our Sparta App

![Screenshot 2023-10-25 101058.png](images%2FScreenshot%202023-10-25%20101058.png)

We want to change the OS Disk Type to `Standard SSD` and click Next

![Screenshot 2023-10-25 101444.png](images%2FScreenshot%202023-10-25%20101444.png)

Select our own 'Virtual Network' and select the public subnet

![Screenshot 2023-10-25 101616.png](images%2FScreenshot%202023-10-25%20101616.png)

Skip Management & Monitoring and we want to add User Data to deploy our Sparta App

![Screenshot 2023-10-25 101857.png](images%2FScreenshot%202023-10-25%20101857.png)

## Troubleshoot

Sudo upgrade gives us this error, which requires a manual intervention so we need to change our `sudo apt upgrade -y` <br>

![MicrosoftTeams-image (3).png](images%2FMicrosoftTeams-image%20%283%29.png)

To: 
```
sudo DEBIAN_FRONTEND=noninteractive apt-get upgrade -y -o Dpkg::Options::="--force-confnew"
```

## Step 2: Paste this script into the User Data:

```
#!/bin/bash

# update & upgrade
sudo apt update -y
sudo DEBIAN_FRONTEND=noninteractive apt-get upgrade -y -o Dpkg::Options::="--force-confnew"

# install nginx
sudo apt install nginx -y

# setup nginx reverse proxy
sudo apt install sed
# $ and / characters must be escaped by putting a backslash before them
sudo sed -i "s/try_files \$uri \$uri\/ =404;/proxy_pass http:\/\/localhost:3000\/;/" /etc/nginx/sites-available/default

# restart nginx 
sudo systemctl restart nginx

#enable nginx

sudo systemctl enable nginx

# install nodejs 12.x
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install nodejs -y

# install pm2 (only necessary later)
sudo npm install pm2 -g

# clone repo with app folder into folder called 'repo' - only needed if don't have the app folder already
sudo apt install git -y

git clone https://github.com/shaluomehra/sparta_test_app.git

# install the app (must be after db vm is finished provisioning)
cd sparta_test_app
cd app
npm install

# start the app (could also use 'npm start')
pm2 start app.js
```

Don't forget to add a Tag to your Virtual Machine

![Screenshot 2023-10-25 102732.png](images%2FScreenshot%202023-10-25%20102732.png)

Then Click Create and Launch!

## Step 3: Create a New VM to Link the Database

This time, we want to use the Private Subnet and no Public I.P, on 'NIC network security group' click 'Advanced' and we need to click 'Create New' to add our MongoDB port

![Screenshot 2023-10-25 111337.png](images%2FScreenshot%202023-10-25%20111337.png)

Click 'Add an inbound rule' and Find 'Mongo DB' under services and this will add the port press 'OK'

![Screenshot 2023-10-25 111612.png](images%2FScreenshot%202023-10-25%20111612.png)

Under User Data we want our updated script:


```
#!/bin/bash

# Update and upgrade packages
sudo apt update
sudo DEBIAN_FRONTEND=noninteractive apt-get upgrade -y -o Dpkg::Options::="--force-confnew"

# Acquire the MongoDB 3.2 key
wget -qO - https://www.mongodb.org/static/pgp/server-3.2.asc | sudo apt-key add -
echo "Acquired MongoDB 3.2 key successfully."

# Add MongoDB repository to sources.list
echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

# Update packages to fetch MongoDB 3.2
sudo apt update

# Install MongoDB 3.2 packages
sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20
echo "MongoDB 3.2 installed successfully."

# Modify MongoDB configuration to allow all connections
sudo sed -i 's/bindIp: 127.0.0.1/bindIp: 0.0.0.0/' /etc/mongod.conf
echo "Modified MongoDB configuration to allow all connections."

# Start MongoDB and enable auto-start
sudo systemctl start mongod
sudo systemctl enable mongod

# Check MongoDB service status
sudo systemctl status mongod
```

**Don't forget to add Tags**

Click 'Create'

## Step 4: Create a new APP VM

We need to edit the user data, so we need to create a new app VM and tweek the user data slightly

We have added an environment variable called 'DB_HOST' this will link the app and the db: <br> `export DB_HOST=mongodb://10.0.3.4:27017/posts`

We have also added `node seeds/seed.js` to seed some random data onto the database
```
#!/bin/bash

# update & upgrade
sudo apt update -y
sudo DEBIAN_FRONTEND=noninteractive apt-get upgrade -y -o Dpkg::Options::="--force-confnew"


# install nginx
sudo apt install nginx -y

# setup nginx reverse proxy
sudo apt install sed
# $ and / characters must be escaped by putting a backslash before them
sudo sed -i "s/try_files \$uri \$uri\/ =404;/proxy_pass http:\/\/localhost:3000\/;/" /etc/nginx/sites-available/default

# restart nginx
sudo systemctl restart nginx

#enable nginx

sudo systemctl enable nginx

# install nodejs 12.x
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install nodejs -y

export DB_HOST=mongodb://10.0.3.4:27017/posts

# install pm2 (only necessary later)
sudo npm install pm2 -g

# clone repo with app folder into folder called 'repo' - only needed if don't have the app folder already
sudo apt install git -y

git clone https://github.com/shaluomehra/sparta_test_app.git

# install the app (must be after db vm is finished provisioning)
cd sparta_test_app
cd app
npm install
node seeds/seed.js

pm2 kill

# start the app (could also use 'npm start')
pm2 start app.js
```

## Useful Information

### How to SSH into an instance

SSH in using 'Native SSH'

![Screenshot 2023-10-25 113314.png](images%2FScreenshot%202023-10-25%20113314.png)

Provide the path to the private key file and then copy the command in grey into GitBash

![Screenshot 2023-10-25 114134.png](images%2FScreenshot%202023-10-25%20114134.png)










