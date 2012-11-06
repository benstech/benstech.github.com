---
layout: post
title: "Powershell: Restart And Resume"
category: Powershell
tags: [Powershell, Beginner, Restart, Resume]
---
{% include JB/setup %}
Sometimes we need to restart the computer and resume executing the script.
This can be done by add the script to the Windows Registry to make it automatically run
after the computer starting on.

If you know something on Windows Registry, you probably know items under 
HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
will run automatically after Windows starting on.
So what we do is to add an item under this path, and make it point to our the path of our script.

But there is one problem: the script will run from the very beginning each time the computer starts.
That's not what we want. We want the script to continue from where it stops last time.
To achieve this, we need to organise our script into several steps. The script goes like this:

{% highlight powershell %}
param([int]$Step = 1)   # For rebooting, indicate which step the script is runing
switch($Step){
	1 {
		Do something...
		Add the script to the registry with parameter ($Step + 1)
		Restart-Computer
	}
	2 {
		Do something...
		Add the script to the registry with parameter ($Step + 1)
		Restart-Computer
	}
	3 {
		...
	}
}
{% endhighlight %}

Notice that we use a parameter here to indicate which step the script is running.
Each time we want to restart, we add the current script path to the registry with
the step value increasing 1. So each time the computer reboots, the script will run
into the next step.

So this is the idea. The code is listed below. I write a function called RestartAndResume, 
which will do the work described above.

{% highlight powershell %}
Function ClearRegRunKey(){
    
    $private:regRunPath = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" 
    $private:regKeyName = "TKFRSAR"

    if (((Get-ItemProperty $regRunPath).$regKeyName) -ne $null)
    {
        Remove-ItemProperty -Path $regRunPath -Name $regKeyName
    }
}

Function RestartAndResume(){
    # If the key has already been set, remove it
    ClearRegRunKey

    $private:regRunPath = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" 
    $private:regKeyName = "TKFRSAR"

    $currentScriptPath = $MyInvocation.MyCommand.Definition

    if($script:step -eq $null){
        throw "Missing variable `$script:step."
    }

    $nextStep = $script:step + 1

    Set-ItemProperty -Path $regRunPath -Name $regKeyName `
                     -Value "cmd /c powershell $currentScriptPath -step $nextStep" `
                     -Force
    # Show prompt
    Write-Host "The computer is going to restart."
    Pause

    # Restart
    Restart-Computer
}
{% endhighlight %}

Function ClearRegRunKey is used to clear the key we add to the registry. This function 
should be called at the end of the whole script. Or else, the script will run automatically
when we login in Windows even afer the script all finished.

And here I use $MyInvocation to get the current script path. I think I need to write a new
post to talk about this variable.

One more thing, usually we want the process to be perfect automatic, but if your computer 
has a password, you need to type in the password after restarting. If you don't want to 
type in the password each time, you can modify the registry in the script to enable auto logon.

{% highlight powershell %}
Function AutoLogon(){
	Set-ItemProperty -path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" `
					 -Name "DefaultUserName" -Value "$domain\$username" -Force
	Set-ItemProperty -path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" `
					 -Name "AltDefaultDomainName" -Value "$domain\$username" -Force
	Set-ItemProperty -path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" `
					 -Name "DefaultPassword" -Value "$password" -Force
}
{% endhighlight %}

*End*
