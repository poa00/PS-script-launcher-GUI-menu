# Use a CSV file to make a graphical menu of PowerShell scripts (Tutorial)
Posted on 6th November 2019 by [Dan (original PWSH Script Menu GUI author)](https://github.com/dan-osull)

I’ve recently been working on a PowerShell module that uses a CSV file to create a custom menu of scripts.

...

It hopefully helps to bridge the gap between engineers and automators, who write scripts useful to others, and service desk people and technicians, who may not be confident with the command line. PSScriptMenuGui allows PowerShell coders to put their scripts in a simple menu, usable by anyone.

It’s loosely inspired by (criminally similar to) something I made for a previous employer.

(The main difference is that this version starts instantly. The old version was so slow that I made an entertaining loading screen to fill the void. A good subject for a future blog post.) If you’d like to dive straight in, please:

1. Install PSScriptMenuGui from the PowerShell Gallery and
2. Head over to GitHub for documentation
3. The rest of this blog post acts as a tutorial.

If you’d like to be guided through making your own menu, keep reading…

## Steps to Setting Up Menu

### Step 0: System requirements

The module works on **Windows only** – sorry, rest of world! Apart from that, it should run pretty much anywhere. It works on:
- PowerShell for Windows 5.1 which comes with Windows 10.
- PowerShell 7, currently available as a preview and due to be finished at the start of 2020.

>- It does **not** work on PowerShell Core 6. If you have this version, the easiest solution is to use PowerShell 5.1 as it is already on your PC.

### Step 1: Install the module and make an example menu

Open a PowerShell prompt and:

```powershell
# Navigate to where you want to work on your menu - in my case OneDrive:
cd $env:OneDrive
# Install the module:
Install-Module PSScriptMenuGui -Scope CurrentUser
# You may need read and agree to messages about updates and trust
# Make an example menu to get you going:
New-ScriptMenuGuiExample
```

You should see this:

```console
VERBOSE: Copying example files to PSScriptMenuGui_example...
```

### Step 2: Explore the example

Navigate to your PSScriptMenuGui_example folder and open PSScriptMenuGui.ps1. You should see a bit of boilerplate to ensure that the module is loaded, followed by this line which displays the menu:

```powershell
Show-ScriptMenuGui -csvPath '.\example_data.csv' -Verbose
```

Try running the line in your PowerShell window. You should see the example menu from the GIF at the top of this post.

Now open `example_data.csv`. A text editor is fine but Excel is easier. You can see that every row in the CSV represents an item in the menu.

### Step 3: Make it your own

Experiment with editing the CSV and running the Show-ScriptMenuGui command again to see your changes.

#### A few ideas:

Put one of your scripts in the folder and add it to the menu using Method `powershell_file` and Command `.\filename_of_script.ps1`.
Try including PowerShell commands in the CSV file. Use Method `powershell_inline` and Command `Get-ComputerInfo`. Run `Show-ScriptMenuGui` with `-NoExit` to stop the PowerShell window from closing.
Add a link to an external application. Use Method cmd and enter the path of the program in Command.

### Step 4: Next steps

Go to GitHub for full documentation. In particular, see the CSV reference.
Check out PSScriptMenuGui_all_options.ps1 in your PSScriptMenuGui_example folder.
Run Get-Help Show-ScriptMenuGui -Full

### Step 5: Make a shortcut
When you’re happy with your menu, why not make a shortcut to it in File Explorer?

- Right click a blank area in a folder window or on your desktop.
- Select New → Shortcut
- Enter something like this as the location:
 ```powershell
  C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -File  "C:\full\path\PSScriptMenuGui_example\PSScriptMenuGui.ps1"
  ```

<br>

***
### Thoughts

#### Posted to Tutorial:

##### Jim Birch says:
6th November 2019 at 11:44 pm
I have a little exe that does something similar. Allows various commands and scripts to run. It’s particularly useful to runas with my different user roles, so I can start admin level apps without multiple clicks and id/password entry.

##### Kevin says:
4th May 2020 at 11:05 am
This menu system is fantastic! So easy to build and run. Now my question is, after I open a PowerShell window from the menu, can I use the menu to run more commands from the open PS shell? For example, I created a menu item to connect to Azure. Now within the Azure shell I would like to run a command. It would be great if I could create another menu item to run my command inside of the window I just opened. Thanks!

#### Dan says:
6th May 2020 at 2:29 pm
Hi Kevin,
Glad you’ve found the module useful and thanks for the interesting question!
There’s currently no built-in way of doing what you’re describing.
It may be possible using a .NET Named Pipe – this blog post is a good intro:
https://rkeithhill.wordpress.com/2014/11/01/windows-powershell-and-named-pipes/
I’d be interested to know how you get on – feel free to drop me a line.

##### Dan M says:
4th November 2021 at 10:50 am
I love this but is there a way to create a CSV file for side by side buttons as my list of buttons is getting rather LONG

#### Posted to the author's GitHub - Issues section:

##### @Rapidhands commented on Dec 13, 2020

I've discovered your module. Great job. Could i suggest you add more samples for use it.
i.e. : Auto build GUI - How ? Always with the .csv file in input,.. but the .csv file could be build automatically.

In a Main Script :

Gather script files (.bat, .cmd, .ps1) in a specific folder (i.e. Scripts )
Building the .csv file automatically with info gathered
**Section** : Auto-completion depending on file name. i.e. If file name begin by "Get", section will be "QUERIES", il the name file begin by "ADD" section will be "NEW", ...
**Method:** Auto-completion depending on file type (.ps1, .cmd, ...)
**Command:** auto-completion depending of file Name excluding the prefix of the file Name
**Descrption:** auto-completion depending of the Help Section in the script
if .cmd or .bat file : read first line (comment) in the file script
if .ps1 file : read .SYNOPSIS section
Saving the .csv file
Call Show-MenuGui cmdlet
With this, anyone could build a dynamic GUI depending of the currents scripts in path

I'm thinking that this could be a real good enhancement to show or describe about your module in your Gibhub

I'm aware that to realize this, there are some prerequisites (Naming convention for script files, Help or comments in script files, ...)

Regards
Olivier

#### @Rapidhands commented on Dec 13, 2020

Something like this:

```powershell
# Gather All Scripts in RootPath 
$AllScripts = Get-ChildItem -Path "$RootPath\Scripts\" -File
#Array Initialization
$Data = @()
#Populate Array
foreach ($Script in $AllScripts)
    {
    $Extension = if  ( $script.extension -eq ".ps1") {"powershell_file"}
                 else{ 
                      if ( ($script.Extension -eq ".bat") -or ($script.Extension -eq ".cmd") )
                        {"cmd"}
                     }
    $ScriptType = ($Script.Name).split("-",2)[0]
    $Synopsis = Get-Content -Path $script.FullName | Select-String -Pattern ".Synopsis" -Context 0,1
    
    # Feed a PSCustomObject
    $objet = [PSCustomObject]@{Section     = $ScriptType
                               Method      = $Extension
                               Command     = $Script.FullName
                               Arguments   = ""
                               Name        = $Script.Name
                               Description = $Synopsis
                               }
    # Feed the FinalArray
    $Data += $objet
    }

# Creating .csv file...
# Path for .csv file
$csvRootPath = "YourPath"
if (Test-Path $csvRootPath\DataForMenu.csv)
    {
    # removing existing .csv file
    Remove-Item -Path $csvRootPath\DataForMenu.csv -Force
    }
$data | Export-Csv -Path $csvRootPath\DataForMenu.csv -Delimiter "," -NoTypeInformation
# Building GUI ...
Show-ScriptMenuGui -csvPath $csvRootPath\DataForMenu.csv
```

What do you think about this . You can add this as sample (MainScript)
I see a user-case for this :

Admin users have no access to the Internet ... (i.e. for secure reason)
You have created a bunch of scripts for your colleagues and copy/paste/sendto ... but they don't know how to use them.
Answer to them : "Just launch MainScript.ps1 at the root on the root folder. This root folder contains /scripts and /Modules (used by some scripts) /functions (used by some scripts)
The GUI (based on your module) shows all essentials info about the scripts they could use
