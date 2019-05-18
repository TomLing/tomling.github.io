---
layout: post
title: How to export VS Code extensions to another computer using PowerShell
categories: [vscode, powershell]
tags: [vscode, powershell, extensions, export, vs]
description: Exporting VS Code extensions to another computer is easy using PowerShell.
comments: true
---

## Setting up the script

VS Code is one of my favourite IDEs for web development and text editing, In my opinion, it is currently the best choice out there if you are looking for a powerful development tool with a rich set of features. One thing lacking from VS Code is the ability to easily transfer your development setup and extensions from one machine to another. In this blog post, I will demonstrate a simple piece of PowerShell that makes it very straightforward.

First, open up the command prompt (you will need PowerShell installed and enabled) then navigate to a directory where you have permission to create files.

Run the following command

{% highlight powershell %}
code --list-extensions | ForEach-Object {"code --install-extension $_"} > extensions.ps1
{% endhighlight %}

This will create a PowerShell script file named extensions.ps1 in your chosen directory.

## Running the script

Now for the final step, simply transfer the created script into a directory on the target machine.

Navigate to the folder containing the script and run it using the following command

{% highlight powershell %}
.\extensions.ps1
{% endhighlight %}

The script will run and install all of the extensions taken from your initial development environment.

If the extensions have customization settings you would like to transfer, you will also need to copy over your settings.json.

This can be achieved by opening VS Code on the initial development environment and holding Ctrl+Shift+P to show a list of available commands. Type "Preferences: Open Settings (JSON)" and hit enter.

This will open a file called settings.json. To transfer your user settings simply copy the contents to the target machine by running the same command to open the file.

That's it. You should now have a development environment that has been transferred to another machine by running a simple PowerShell script.