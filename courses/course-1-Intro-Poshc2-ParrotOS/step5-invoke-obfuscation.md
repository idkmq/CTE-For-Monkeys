# Powershell Obfuscation 

```
" Adversaries and attackers relies on the Obfuscation techniques to hide their malicious payload and to avoid detection of their attack. Even with logging enabled, security teams might struggle to identify the attack patterns and the indicators of the attack.

Invoke-Obfuscation is a tool developed to aid Blue Teams to simulate obfuscated payloads and to enhance their detection capabilities. This tool helps security teams to adapt the techniques used by adversaries and to find malicious indicators. " - Ammad Alli
```

1. First lets download the `invoke-obfuscation` from github to our Parrot Vm
    ```git 
    git clone <repository-url>
    ```
    > **NOTE:** Go to [Invoke-obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation) Github click on `Code` copy the `HTTPS` link 

2. Now that we have downloaded `invoke-obfuscation` lets cd to the new folder and start our powershell instance 
    ```bash
    cd Invoke-obfuscation
    ```
    ```bash 
    pwsh
    ```
3. Now in our powershell shell we invoke our powershell module we want to use
    ```powershell 
    Import-Module .\Invoke-Obfuscation.psd1
    ```
4. Lets check the steps by running tutorial 
    ```
    tutorial 
    ``` 
5. for this tutorial lets use a simple command to show off what will happen, we will use the following command 
    ```powershell
    write-output 'I hate it here, please send help.'
    ```
6. To start lets set the command as our `scriptblock` 
    ```powershell 
    set scriptblock " write-output 'I hate it here, please send help.'"
    ```
7. Now lets run it through `encoding` and choose `hex/2`
    ```console 
    $ encoding 
    $ 2
    ```
8. You can see what our command looks like now under `result`
9.  Now lets add some `compression` to it
    ```console 
    $ compress
    $ 1
    ```
    You should be seeing your script changing in the `result` output   

10. Now we can test our newly obfuscated script by running `test` or pasting it in a powershell terminal 
    ```powershell 
    test
    ```
Now that you have ran used this tool, write something else or start over and apply different option to see how it affects your script. keep in mind that depending what you stack up it could break your code. If so you can run `undo` to remove the last applied action. 

If you dont already see it the application of this can be used with your powershell download or a reverse shell, making it harder for the blue team to figure out your actions. 

# Resources 
- [Invoke-obfuscation Github Page](https://github.com/danielbohannon/Invoke-Obfuscation)
- [Invoke-obfuscation Tutorial](https://medium.com/@ammadb/invoke-obfuscation-hiding-payloads-to-avoid-detection-87de291d61d3)