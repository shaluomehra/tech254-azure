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


## Blockers

Since the app is running on the 'root' user and we want to run it on the admin user we have to kill the app on the 'root' user first

Using `ps aux | grep pm2`

And then `sudo kill <id>` and we are ready to start the app on admin user






















