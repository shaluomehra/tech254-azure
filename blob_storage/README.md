# Blobs

## Getting Azure CLI

Load up the Sparta APP using User Data and then SSH into Azure Virtual Machine <br>

Use `cd /` to get to the admin user (where the app is located)
and from here you can go in to the app: <br> 

`cd sparta_test_app` <br>
`cd app`

> **Note**:  to see all the processes running we can use the command 
`pm2 aux`

Now we want to install Azure CLI using the command
```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

Once installed we want to **log in** to Azure to use the CLI using `az login` <br>

You will receive instructions on your Git Bash terminal, follow the steps. 

Once you're in you'll see something like this: 

![Screenshot 2023-10-26 162821.png](..%2Fimages%2FScreenshot%202023-10-26%20162821.png)

## Create a Storage Account 
```
az storage account create --name tech254shaluostorage --resource-group tech254 --location uksouth --sku Standard_ZRS`
```

## Create a Container

We're creating a container and calling it test container

```
az storage container create \
     --account-name tech254shaluostorage \
     --name testcontainer \
     --auth-mode login
```

**Create a test file to upload to the container**

`nano test.txt` and input the word `test`

Use this code to upload the test file
```
az storage blob upload \
     --account-name tech254shaluostorage \
     --container-name testcontainer \
     --name newtest.txt \
     --file test.txt \
     --auth-mode login
```
Search for Storage accounts on Azure

![Screenshot 2023-10-27 164529.png](..%2Fimages%2FScreenshot%202023-10-27%20164529.png)

Click on your Storage folder

![Screenshot 2023-10-27 164545.png](..%2Fimages%2FScreenshot%202023-10-27%20164545.png)

Click on 'Containers' then you can see your test container that we created earlier
![Screenshot 2023-10-27 165101.png](..%2Fimages%2FScreenshot%202023-10-27%20165101.png)

We need to change the 'Access Level' to 'blob' so we can access the Public I.P

![Screenshot 2023-10-27 165311.png](..%2Fimages%2FScreenshot%202023-10-27%20165311.png)

Now if you go into the `testcontainer` we can see our test file and if you follow the URL you can see the text on the web page



## Blockers

Since the app is running on the 'root' user and we want to run it on the admin user we have to kill the app on the 'root' user first

Using `ps aux | grep pm2`

And then `sudo kill <id>` and we are ready to start the app on admin user






















