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








