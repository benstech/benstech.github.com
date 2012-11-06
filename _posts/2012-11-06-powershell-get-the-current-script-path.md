---
layout: post
title: "Powershell: Get The Current Script Path"
category: Powershell
tags: [Powershell, Beginner, Current, Script, Path, MyInvocation]
---
{% include JB/setup %}
There is a global variable named MyInvocation which can provide information 
on which script the powershell is running.
If you want to get the script name, use

{% highlight powershell%}
$scriptName         = $MyInvocation.MyCommand.Name
{% endhighlight %}

And if you want to get the full path of the current script, use

{% highlight powershell%}
$scriptPath         = $MyInvocation.MyCommand.Definition
{% endhighlight %}

MyInvocation is actually a property of PSCmdlet class. I don't quite
understand the mechanizm. For more information, 
refer to [here](http://msdn.microsoft.com/en-us/library/windows/desktop/ms551396(v=vs.85).aspx).
