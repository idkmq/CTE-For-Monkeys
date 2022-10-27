# Installing NGINX on Ubuntu

1. Update and install command 
    ```bash 
    sudo apt update
    sudo apt install nginx
    ```

# Aduj Firewall - might not need
1. View new applications
    ```bash 
    sudo ufw app list
    ```
2. Allow nginx HTTP
   ```
   sudo ufw allow 'Nginx HTTP'
   ```
3. Verify changes 
    ```bash 
    sudo ufw status
    ```

# Checking out Nginx Server 

1. Check your Nginx Server is running 
    ```
    sudo systemctl status nginx
    ```
2. Browse to your nginx landing page insure its working.


# Create Your own Domain 

1. Make your domain folder
   ```
   sudo mkdir -p /var/www/your_domain/html
   ```
2. give permissions to new folder 
   ```
   sudo chown -R $USER:$USER /var/www/your_domain/html
   ```
3. add web root permissions 
   ```
   sudo chmod -R 755 /var/www/your_domain
   ```
4. Create new simple index.html
   ```
   sudo vi /var/www/your_domain/html/index.html
   ```
5. Paste simple HTML
    ```html
    <html>
        <head>
            <title>Welcome to your_domain!</title>
        </head>
        <body>
            <h1>Success!  The your_domain server block is working!</h1>
        </body>
    </html>
    ```
6. In order for Nginx to serve this content, it’s necessary to create a server block with the correct directives.
    ```bash
    sudo vi /etc/nginx/sites-available/your_domain
    ```
7. Paste following config block 
    ```nginx
    server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
    }
    ```


# Manging The NGINX Process 

Now that you have your web server up and running, let’s review some basic management commands.

1. To stop your web server, type:
    ```bash
    sudo systemctl stop nginx
    ```
2. To start the web server when it is stopped, type:
    ```bash
    sudo systemctl start nginx
    ```
3. To stop and then start the service again, type:
    ``` bash
    sudo systemctl restart nginx
    ```
4. If you are only making configuration changes, Nginx can often reload without dropping connections. To do this, type:
    ```bash
    sudo systemctl reload nginx
    ```
5. By default, Nginx is configured to start automatically when the server boots. If this is not what you want, you can disable this behavior by typing:
    ```bash
    sudo systemctl disable nginx
    ```
6. To re-enable the service to start up at boot, you can type:
    ```bash
    sudo systemctl enable nginx
    ```

You have now learned basic management commands and should be ready to configure the site to host more than one domain.


# Resources
- [How to setup NGINX server on ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04)
  - > This writeup was made following this if you need a better explination
- [Something forwarding C2 coms](https://coffeegist.com/security/resilient-red-team-https-redirection-using-nginx/)