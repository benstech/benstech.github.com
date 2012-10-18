---
layout: post
title: "Powershell: File Comparing"
category: Powershell 
tags: [powershell]
---
{% include JB/setup %}

First post is about powershell, since I am new to powershell. 

I wrote this script, because I was working on a C# project consisting of lots of source files. 
I modified some source files, but I couldn't remember. 
To open up every files to see whether I made modifications is time-consuming. 
So I need a script to do that for me.

I'll first introduct the required knowledge on powershell in order to finish the script, 
then put out the whole script.

## Requirements
Now we have two folders: one is the old folder which of course contains all the original files,
and the other is the new folder which contains the modified files.

What we need to do is:
- go through all the files in the old folder
- for each file, pick its corresponding file in the new folder (suppose no files deleted or added)
- compare the old file with the new file
- take down whether the file is changed or not; if changed, write the change text to a log file.

## Powershell Cmdlets

**Get-ChildItem**
The first challenge is how to get all files from a folder. 
This can be done by a very basic cmdlet named Get-ChildItem.

**Others**

{% highlight powershell %}
[string]$oldFile = Get-Item ./
{% endhighlight %}

## Powershell Code
The full powershell is below. The two folder paths are passed as parameters.

	param(
	[string]$oldFolder = ".\Old",
	[string]$newFolder = ".\New",
	[string]$difLogPath    = ".\Dif.log"
	)

	$oldFileList = Get-ChildItem $oldFolder | where {$_.Extension -eq ".cs"}
	$newFileList = Get-ChildItem $newFolder | where {$_.Extension -eq ".cs"}

	$changedFileList = @()
	$unchangedFileList = @()

	New-Item $difLogPath -ItemType File -Force

	foreach ($oldFile in $oldFileList){
	    
	    $newFile = Get-ChildItem $newFolder | where {$_.Name -eq $oldFile.Name}
	    
	    $oldFileContent = Get-Content $oldFile.FullName
	    $newFileContent = Get-Content $newFile.FullName
	    
	    Write-Host "Checking $oldFile ..."

	    $dif = Compare-Object -ReferenceObject $oldFileContent -DifferenceObject $newFileContent

	    if ($dif -eq $null){
		$unchangedFileList += @($oldFile.Name)
	    }
	    else{
		$changedFileList += @($oldFile.Name)
		$oldFile.Name | Out-File -FilePath $difLogPath -Append
		$dif | Out-File -FilePath $difLogPath -Append
	    }
	   
	}


	Write-Host "`r`nChanged Files are:" -ForegroundColor Yellow
	foreach ($item in $changedFileList){
	    Write-Host $item -ForegroundColor Red
	}

	Write-Host "`r`nUnchanged Files are:" -ForegroundColor Yellow
	foreach ($item in $unchangedFileList){
	    Write-Host $item -ForegroundColor Green
	}
