
# Create a Freash Debian 12 server on DigitalOcean
---
# Creating a Debian server
## Creating ssh key pair
Open your terminal
Create a new .ssh directory
`mkdir .ssh`

Create a ssh public/private key pair (replace example command with your own actual email address)
`ssh-keygen -t ed25519 -f .ssh/do-key -C <your-email-address>`

Copy your public key in the .ssh folder to the clipboard to connect to Digital Ocean (replace with correct "username" and file path)
`Get-Content C:\Users\<user-name>\.ssh\do-key.pub | Set-Clipboard`
## Digital Ocean Droplet
Connect your ssh key to DO
- Settings -> Security -> "Add SSH Key"

Create a new droplet in the main Dashboard
- Set server to San Francisco SFO3
- OS = Debian
- $7/ month payment plan
- use the SSH key to login

## Connect to your server
Connect to your server from terminal while in the .ssh folder
`ssh -i <path-to-key> root@<droplet ip address>`
example: `ssh -i do-key root@143.198.106.177`

Type yes to enter your server

# Replace logging in as the root user
Working as the root user is bad practice and can cause security users

## Creating a regular user
Currently logged in a root user
To create a new user (DO NOT NAME USER)
`useradd -ms /bin/bash <username>`
-m will create a home directory for the new user
-s will set the users path to /bin/bash

Create a password for your user
`passwd <username>`

## Adding a regular user to the sudo group
We still want to access sudo commands as our user when admin requirements are need
`usermod -aG sudo <username>`
-a will append the group instead of replacing
-G is the group 

Check if sudo is included in the groups
`groups <username>`

## Give the ability to login directly as user
Switching from root to "user"
`su -l <username>`
-l simulates a full login

Copy the contents of the .ssh folder to our user's home directory
`sudo cp -r /root/.ssh /home/<username>`
-r recursively copies the contents
Check if the .ssh folder is there
`ls -la`
it is still owned by root (`root root`) 

Change the file ownership of .ssh from root to regular user
`sudo chown -R <username>:<username> /home/<username>/.ssh`
-R recursively changes everything in directory
Use `ls -la` and see that the owner and group are changed

Test if you can log in as the new user
`logout` out of debian
`ssh -i <path-to-key> <username>@<droplet ip address>`

## Remove root login privilege's 
Since we can log in as our regular user we will remove the ability for root to login
while in `home/<username>` nagivate to the ssh folder
`cd /etc/ssh`

We will be changing the login settings in sshd_config
`sudo vim sshd_config`
find `PermitRootLogin yes` 
change `yes` to `no` 

Restart the ssh service to load our new settings
`sudo systemctl restart ssh.service`

`logout` and check if you can log in as root user.
you should get `Permission denied (publickey).`




# Configure nginx to serve a sample website
## Install and enable nginx
Ensure your packages are up to date
`sudo apt update`

Install nginx
`sudo apt install nginx`

Check if it is enable and running
`systemctl status nginx`

If disabled, then you need to enable
`sudo systemctl enable --now nginx`

## Change the default html to be served
Create a new directory in the default folder
```
cd /var/www
sudo mkdir my-site
```

create a new index.html file in my-site
`sudo vim index.html`
input the follow html code and save
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>

```

## Creating a new server block
Create a new .conf file in `/etc/nginx/sites-available`
`cd /etc/nginx/sites-available`
`vim my-site.conf`

Input and save the following server block
```
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	
	root /var/www/my-site;
	
	index index.html index.htm index.nginx-debian.html;
	
	server_name _;
	
	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}
}
```

Unlink the default so only our new block will be registered when called
`sudo unlink default`

Create a symbolic link to our new config file in `etc/nginx/sites-enabled`
`cd etc/nginx/sites-enabled`
`sudo ln -s /etc/nginx/sites-available/my-site.conf my-site.conf`

Unlink the default so only our new block will be registered when called
`sudo unlink default`

Restart the nginx service to load the new changes
`sudo systemctl restart nginx`

Check if the our new html is displayed
`curl <droplet ip address>`

Check ip address if forgotten using `id addr show` 
It will be the one under "eth0" after the first "inet"
