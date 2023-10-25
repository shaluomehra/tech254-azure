## Step 1: Creating new SSH Key Pair
1. Open GitBash terminal
2. go into SSH folder
3. Create key pair 0- `ssh-keygen -t rsa -b 4096 -C "shaluomehra@outlook.com"`

## Step 2: Login to Azure
1. Go to Azure portal.azure.com
2. Login with Sparta Global email and login details

## Step 2b: Link SSH Key Pair
3. Search SSH Keys in search bar and click on SSH KEYS
4. Click Create
5. Subscription  = Azure Training
6. Resource Group = Tech254
7. Region done automatically = (Europe (UK SOUTH))
9. SSH Public Key Resource > Upload existing public key
10. Upload key > Copy and Paste the PUBLIC KEY
11. Click Tabs
12. Name = Owner, Shaluo
13. Click Review + Create

## Step 3: Create Virtual Network
1. Search Vnet
2. Click Create
2. Subscription is automatically filled in = Azure Training
6. Resource Group = Tech254
7. Virtual Network Name = **tech254-shaluo-app-db-vnet
8. Region = (Europe) UK South 
9. Click Next
*NOTE: Security Page left alone. Nothing ticked*
10. Click Next to IP Addresses
11. Click the pen/pencil edit button and a popup should appear
12. Edit Name to public-subnet
13. Starting address = `10.0.2.0/24`
14. Press Add
15. Add Subnet 
16. Edit Name to private-subnet
13. Starting address = `10.0.3.0/24`
14. Press Add
16. Next to Tags
17. Name = Owner
18. Value = Shaluo

## Step 4: Create Virtual Machines
1. Search Virtual Machine
2. Click Virtual Machine
3. Click Create
4. Virtual machine name = `tech254-shaluo-first-app-vm`
5. Image = Search for Image = `Ubuntu Pro 18.04 LTS - x64 Gen2`
6. Security Type = Standard
7. Size = Standard_B1s -1 vpcus
8. Username = adminuser
9. SSH Public key source > Use existing key stored in Azure
10. Stored keys = `tech254-shal-az-key`
11. Click HTTP and SSH for Security Inbound Rules
11. Click Next to DISK
12. Select Standard SSD
13. Next To Networking
14. Skip Management and Monitoring 
15. Go to Advanced
16. Copy and paste script to User Data:

```
#!/bin/bash

# update & upgrade
sudo apt update -y
sudo apt upgrade -y

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