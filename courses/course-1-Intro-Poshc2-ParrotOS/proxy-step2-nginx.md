# Setting up proxy using Ubunut and Nginx

When running actions as an actor you always want to avoid giving away where you are coming from or your source ip. Typically this is done with a proxy also sometimes known as a LP (listening post), in this case it will be an ubuntu server running Nginx. Your current setup is as follows: 

	parrot-OS <> Victim VM

What we will be doing with the Ubuntu server is routing our C2 traffic through it so it appears it as though it is coming from the Ubuntu server and the the Parrot vm.

	Parrot <> Ubuntu <> Victim VM 

Lets install any Ubuntu server ISO from the interwebs [here](https://ubuntu.com/download/server).

# Install Nginx


## Important Nginx File Locations 
- `/var/www/html` – Website content as seen by visitors.
- `/etc/nginx` – Location of the main Nginx application files.
- `/etc/nginx/nginx.conf` – The main Nginx configuration file.
- `/etc/nginx/sites-available` – List of all websites configured through Nginx.
- `/etc/nginx/sites-enabled` – List of websites actively being served by Nginx.
- `/var/log/nginx/access.log` – Access logs tracking every request to your server.
- `/var/log/ngins/error.log` – A log of any errors generated in Nginx.

