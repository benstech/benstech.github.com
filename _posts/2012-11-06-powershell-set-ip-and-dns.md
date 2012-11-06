---
layout: post
title: "Powershell: Set IP and DNS"
category: Powershell
tags: [Powershell, Beginner, IP, Gateway, DNS, Netsh, WMI, Get-WmiObject]
---
{% include JB/setup %}
## Using Netsh ##
**Netsh** is a command-line scripting utility that allows you to display
or modify the network configuration of a computer that is currently running. 
(Refer to [MSDN](http://technet.microsoft.com/en-us/library/cc732279(v=ws.10).aspx))

You can set your IP using the following command:

{% highlight powershell %}
netsh interface ip set address name="Local Area Connection" static 192.168.0.10 255.255.255.0 192.168.0.1 1
{% endhighlight %}

- *Local Area Connection* is the name of the adapter you want to modify.

- *192.168.0.10* is the IP address you want to set. 

- *255.255.255.0* is the subnet mask. 

- *192.168.0.1* is the gateway.

- *1* at the end is the gateway metric. I don't know what it is for, but you can leave this as 1 for almost all cases.  

If you want to enable DHCP you can run:

{% highlight powershell %}
netsh interface ip set address name="Local Area Connection" dhcp
{% endhighlight %}

To configure DNS, use the following commands:

{% highlight powershell %}
# For the primary DNS run:
netsh interface ip set dns name="Local Area Connection" static 192.168.0.1
# For the secondary run:
netsh interface ip add dns name="Local Area Connection" 192.168.0.2 index=2
{% endhighlight %}

To configure the computer to use DNS from DHCP run:

{% highlight powershell %}
netsh interface ip set dnsservers name="Local Area Connection" source=dhcp
{% endhighlight %}

## Using WMI ##

What is WMI? Well, this question confuses me for quite a long time. I found that 
the more I looked at Microsoft MSDN, the more confusing I got.
In my understanding, for programmers, we can just regard WMI as a set of APIs which
provide us ways to access system configurations.

I'd like to copy a paragraph from MSDN:

Windows Management Instrumentation is a core Windows management technology; 
you can use WMI to manage both local and remote computers. 
WMI provides a consistent approach to carrying out day-to-day management tasks with programming or scripting languages. 
For example, you can:

- Start a process on a remote computer.
- Schedule a process to run at specific times on specific days.
- Reboot a computer remotely.
- Get a list of applications installed on a local or remote computer.
- Query the Windows event logs on a local or remote computer.

The word Instrumentation in WMI refers to the fact that WMI can get information about the internal state of computer systems, 
much like the dashboard instruments of cars can retrieve and display information about the state of the engine. 
WMI instruments by modeling objects such as disks, processes, or other objects found in Windows systems. 
These computer system objects are modeled using classes such as Win32_LogicalDisk or Win32_Process; 
as you might expect, the Win32_LogicalDisk class models the logical disks installed on a computer, 
and the Win32_Process class models any processes currently running on a computer.

In powershell, you can use the **Get-WmiObject** Cmdlet to get the specific WMI object.
All the WMI objects can be found by executing:

{% highlight powershell %}
Get-WmiObject -List
{% endhighlight %}

Now let's turn to how to change our IP setting via WMI.
We need Win32_NetworkAdapterConfiguration object to do this.
The code is below:

{% highlight powershell %}
Function SetNetwork($ip,$mask,$gateway,$dns){

    $netObj = Get-WmiObject Win32_NetworkAdapterConfiguration
    $netObj.EnableStatic($ip,$mask)
    $netObj.SetGateways($gateway)
    $netObj.SetDNSServerSearchOrder($dns)
}
{% endhighlight %}

*End*
