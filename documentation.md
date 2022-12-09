#Deploy the Node.Js App to Nginx Server

##Specification - Ubuntu’s latest version – 20.04.

Now, you are required to have an SSH to your server while using Hostinger. You will be able to see the server’s IP address on the right-hand side. You can keep SSH as root.
- `ssh root@42.35.40.01`

##Install MySQL Database
Its installation process is the same as other Debian (Ubuntu) packages throughout the apt.

- `sudo apt update`
- `sudo apt install mysql-server`

Now configure the MySQL Deamon server.
- `sudo mysql_secure_installation`

Don’t forget to check whether the MySQL server is running or not with the given command.
- `systemctl status mysql`

Now, connect the server with the MySQL client.
- `mysql -u root`

Now, we have successfully configured the Database Server without any errors.
- `mysql quit`

##Nginx Reverse Proxy
Here, we use Nginx reverse proxy to receive multiple requests from clients and serve it to different servers. We will use the Nodejs server here.

Now is the time to install Nginx.

- `sudo apt update`
- `sudo apt install nginx`

Now, apache2 is installed with Hostinger. It is fully configured to server HTTP on PORT 80. Hence, we have to disable apache2 from our server.
- `sudo apt-get remove apache2*`

So, let’s install ufw (uncomplicated firewall).
- `sudo apt install ufw`

To access your server in the future, you have to ensure that all the SSH connections pass through the firewall only.
- `ufw allow ssh`

You have to write a given code to check which network application is available and has access through the firewall.
- `ufw app list`
>  Available applications:
Nginx Full
Nginx HTTP
Nginx HTTPS
OpenSSH

Nginx Full: Opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic)
Nginx HTTP: Opens only port 80
Nginx HTTPS: Opens only port 443
Since we need HTTP through ufw, let’s allow Nginx.

Allow the Nginx HTTPS or the Nginx Full.

- `sudo ufw allow 'Nginx HTTP'`

check the Nginx status.
- `sudo systemctl status nginx`

Keep the Nginx status active. If it has not started automatically, then do it manually.
- `sudo systemctl start nginx`

After completing this, we have to set up Nginx Server Block. From our given servers or multiple independent domains, you can serve subdomains.

Nginx Configuration address: /etc/nginx/
Nginx Main Server Root address: /var/www/
Now, let’s create the main IP address or domain serving page root.

- `sudo mkdir -p /var/www/servername.com/html`

Make sure the permission of web root should be correct with the given code.
- `sudo chmod -R 755 /var/www/servername.com`

Here, -R (recursive) will define all servername.com sub-directories having the same permission as the parent directory.

Then, write a given code. This code will assign the ownership of the directory having the existing user named $USER environment variable:
- `sudo chown -R $USER:$USER /var/www/example.com/html`

Now, let’s make a simple index.html webpage for testing purposes. Then integrate the proxy of the Nginx server to our localhost running the NodeJs app.

- `nano /var/www/servername.com/html/index.html`

Write the below-given HTML code inside.


    <html>

     <head> 
        <title>Welcome to servername.com!</title> 
    </head> 
 
    <body> 
        <h1> Success! The servername.com server block is working successfully!</h1>
        <b>Meow Meow!</b>

    </body>
    </html>


Now close the Nano text editor after saving.

Moreover, we should add configuration at /etc/nginx/sites-available for Nginx to serve this content.

- `sudo nano /etc/nginx/sites-available/servername.com`

You should keep the config file the same as your domain name to be easy for you to maintain.

For a simple Nginx configuration, here is the code.

    server {
    listen 80;
    listen [::]:80;

        root /var/www/servername.com/html; 
        index index.html index.htm index.nginx-debian.html; 
 
        #Here, you can put your domain name for ex: www.servername.com 
 
        server_name 42.35.40.01; 
 
        location / { 
                try_files $uri $uri/ =404; 
        } 
    } 

/etc/nginx/sites-available is only available for storing your configuration. If you have to make it active, create a link to the config to the sites-enabled directory to see the effect on the server.

- `/etc/nginx/sites-available is only available for storing your configuration. If you have to make it active, create a link to the config to the sites-enabled directory to see the effect on the server.`

If you want to check errors in the new add config, run the below code.
- `nginx -t`

Now, restart the Nginx for the new config for the modification.
- `sudo systemctl restart nginx`

Now, try to browse the address of the server’s IP or domain name. You will be able to view the test page of the sample webpage served back to you. So, the Nginx server is running successfully behind the ufw firewall.

##Install Node

Here, we will use NVM, which you can download from GitHub. Generally, you can install Node from the apt package manager, but it will not let you install the latest version.

- `sudo apt install wget`

- `wget -qO-https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash`

It will consider the install.sh script and execute it with bash to install NVM.

To apply environment changes, you need to run the .profile or .bashrc from your home folder. And this completely depends upon your requirements and configuration.

- `source ~/.bashrc`
- `source ~/.profile`

Now run nvm command. It will run like a charm.

- `nvm -v`

Now install Node and NPM using NVM with the following code.

- `nvm install 11.0`

Then Node and NPM commands will be available across the command line.
- `node -v`

##Install PM2 and Cloning Repo

PM2 is a NodeJs process manager. It tracks your running Node process to make a debug easier. Also, it shows the log files of running apps.

Let’s install it.

- `npm install pm2 -g`

It’s very easy to install, like the PM2 process.

Now, there is a requirement of the Node.Js app’s code on the server. So, we will use Git Version Control to push and then clone the repository.

- `git add .`
- `git commit -m "Initial Commit"`
- `git remote add origin "https://github.com/username/your_repo"`
- `git push origin master`

Now clone the repository with the given code.
- `mkdir repos`
- `cd repos`
- `git clone https://github.com/username/your_repo`

Suppose the framework is based on a simple Node.Js or Express server with app.js, then you need to use PM2 to start the process.

- `cd /repos/your_repo`

Use PM2 now.

- `pm2 start server app.js`

Here, we will use the app.js instance with PM2. We named it “server” so that we can easily identify our application.

Once you are done with this process, you will get a table showing the running/stopped apps and CPU usage.

- `pm2 ls`

##Configure Reverse Proxy

The Nginx configuration is already added with a simple index.html page.

Now bind it to a localhost address, representing the address of the existing running NodeJs app.

This is where an Nginx takes its place for a reverse proxy.

Now open the servername.com config and do essential changes to work as a reverse proxy.

    server
    
    {
    listen   80;
    server_name 42.35.40.01;
    
            location /  
    
    {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $host;
                proxy_set_header X-NginX-Proxy true;
                proxy_pass http://localhost:3000/;
                proxy_redirect http://localhost:3000/ https://$server_name/;
            }
    }


Once you implement the above code, all your questions from Nginx will be redirected to http://localhost:3000. And that’s the place where you will find the NodeJs server running.

Now change the host and port of your existing NodeJs app.

The provided custom headers will be sent with the proxied request to our NodeJs servers - $remote_addr and $host. These represent the real IP address of those users who raised the requests initially.

Finally, everything is covered, and the setup is complete.

You have deployed a NodeJs application on the server.

##Auto startup after restarting the server (pm2)

###Generate PM2 Start Script for Init System

PM2 is designed to work with the default init system on a Linux system (which it can auto-detect) to generate the startup script and configure PM2 as a service that can be restarted at system boot.

- `pm2 startup`

The startup sub-command tells PM2 to detect available init system, generate configuration and enable the startup system.

You can also explicitly specify the init system like so:

- `pm2 startup systems`

To confirm that the PM2 startup service is up and running under systemd, run the following command (replace the pm2-root.service with the actual name of your service, check the output of the previous command):

- `systemctl status pm2-root.service`

###Start Node.js Applications/Processes (ignore this step as node js has already been started)

Next, you want to start your Node.js applications using PM2 as follows. If you already have them up and running, started via PM2, you can skip this step:
- `cd /var/www/backend/api-v1-staging/`
- `pm2 start src/bin/www.js -n api-service-staging`

Next, you need to register/save the current list of processes you want to manage using PM2 so that they will re-spawn at system boot (every time it is expected or an unexpected server restart), by running the following command:

- `pm2 save`

###Verify PM2 Auto Starting Node.js Apps at Boot

- `pm2 ls`
    or
- `pm2 status`

Note that you can manually resurrect processes by running the following command:

- `pm2 resurrect`

Disable the Startup System

- `pm2 unstartup`
    or
- `pm2 startup systemd`
