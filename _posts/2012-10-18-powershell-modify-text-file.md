---
layout: post
title: "Powershell: Modify Text File"
category: Powershell
tags: [Powershell, Beginner, File, Get-Content, Set-Content, System.IO.File]
---
{% include JB/setup %}
Sometimes we need to modify a bunch of files, and a script can do this very easily.
It is very basic techniques, but very useful.

Now let's see how to do this with Powershell.
I'll introduce two methods: using cmdlets and using file operations.

### Cmdlets ###
First I'll use two basic cmdlets, almost used everywhere in Powershell.

**Get-Content**: This cmdlet, of course, is used to get the content of an object. 
I use this cmdlet to get the content of a text file. If not specified, the cmdlet will return a string type. 
You can use this cmdlet by simply pointing out the file path.
{% highlight powershell %}
$content = Get-Content ".\text.txt"
{% endhighlight%}
Please refer to the help for more details.
You can force the return type to be other types. Like this:
{% highlight powershell %}
$XmlFile = [XML] Get-Content ".\config.xml"
{% endhighlight %}
By doing this you can get a System.Xml typed variable.

**Set-Content**: This cmdlet is used to write contents to a file. 
Pass a file path and a content value to write the content to the file.
{% highlight powershell %}
Set-Content -Path ".\text.txt" -Value "Hello World"
{% endhighlight %}

### Example 1 ###
Here is the example of using Get-Content and Set-Content to modify a text file.

{% highlight powershell %}
param($file)
attrib -R $file
$content = Get-Content $file
$content = $content.Replace("a","b")
Set-Content -Path $file -Value $content
attrib +R $file
{% endhighlight %}

The code above actually replaces all the letter "a" in the text with letter "b".

But what are these two doing here?
{% highlight powershell %}
attrib -R $file
attrib +R $file
{% endhighlight %}
*attrib* is a command which allows a user to change the properties of a specified file.
*-R* means to remove the read-only property of the file, while *+R* sets the read-only property.
To set the file writable is good habit before modifying the file.

### System Class ###
Another way to do this is by using the system file class.
The code is very similar with file operations in C#. 
First use [System.IO.File] static method OpenText to open the file, and read it to a variable.
Modify it and then write back to the file by using a StreamWriter.
Here is an example of this.

### Example 2 ###
{% highlight powershell %}
$read = [System.IO.File]::OpenText($file)
$text = ($read.ReadToEnd()).Trim("`r`n")
$read.Close()
$stream = [System.IO.StreamWriter] $file
$stream.Write($text)
$stream.Close()
{% endhighlight %}

The above code removes all the newline charaters in the file.

*End*
