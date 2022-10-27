# Nginx Reverse Proxy Config 

1. Start you nginx
2. go to /etc/nginx/sites-enabled/yourwebsiteORdefault
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
                proxy_pass https://c2-ip-address;
                proxy_redirect off;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```
> **NOTE:** Set your listening ports for NGINX server and your C2 ip to redirect to.

4. test your config 
    ```bash
    sudo nginx -t
    ```
5. If no error, restart you Nginx server
    ```bash
    sudo systemctl restart nginx.service
    ```
6. do the thing