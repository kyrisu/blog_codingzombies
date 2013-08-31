---
layout: post
title: How to put your Microsoft SQL Server database objects into a GIT repository?
categories:
- coding
tags:
- git
- mssql
- powershell
- version control
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
  _edit_last: '1'
  _social_aggregated_ids: a:2:{s:7:"twitter";a:0:{}s:8:"facebook";a:0:{}}
  _social_aggregation_log: a:1:{i:1358333481;O:8:"stdClass":1:{s:6:"manual";b:0;}}
  _wp-svbtle-kudos: '1'
comments: true
---
Recently I have found a new table and a new procedure in a production database of one of our clients. The problem was - we didn't put it there and the client, even though he is obligated by the agreement to report any changes to the database, didn't report it either. Some of our workflows and processes depend on the assumption that we know the exact state of the database - in this case we did not. Lets assume for a second that we make a change that breaks  the dependencies of the procedure or a table. Then the procedure, that we know nothing about, tries to execute and corrupts some data. Based on the maintenance agreement with the client it will be our fault if we don't have a proof that it is otherwise.

<strong>The problem.</strong>

I thought about the problem for a second and the obvious answer that came to my mind was to use GIT (or actually any other version control system). I had the database in my local repository anyway so why not enforce the developers on the client side to do the same. Unfortunately the problem with this approach is that it would need to go through entire chain of command, with lots of  explanation about benefits, risks and possible disaster scenarios  (? O.o) on every step. Even it it somehow goes through (and it probably won't because there's and obvious cost of implementation and time) - I doubt client developers, or better call them MS SQL administrators who write code, will comply with the policy. Bottom line - to much work, to little benefits. So how do you version MS SQL database? <a title="RedGate" href="http://www.red-gate.com/products/sql-development/sql-source-control/" target="_blank">RedGate</a> has some great tools but with price starting at $395  I'm not really sure if  I would be able to convince my boss to buy us the license - especially that I was pretty sure I'm able to code something like this myself ;) - I'm a developer after all.

<strong>The solution.</strong>

After few minute of searching I've found great <a title="post" href="http://blogs.technet.com/b/heyscriptingguy/archive/2010/11/04/use-powershell-to-script-sql-database-objects.aspx" target="_blank">post</a> by Aaron Nelson ( <a href="https://twitter.com/#!/sqlvariant">@SQLvariant</a> ) on Technet which contained exactly what I needed - a function in powershell that can script database objects into separate files. We will use a slightly modified version of the script (for explanation please refer to the source article):
{% highlight powershell %}
function global:Script-DBObjectsIntoFolders([string]$server, [string]$dbname, [string]$path){
[System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.SMO") | out-null
    $SMOserver = New-Object ('Microsoft.SqlServer.Management.Smo.Server') -argumentlist $server
    $db = $SMOserver.databases[$dbname]

$Objects = $db.Tables
$Objects += $db.Views
$Objects += $db.StoredProcedures
$Objects += $db.UserDefinedFunctions

#Build this portion of the directory structure out here in case scripting takes more than one minute.
$SavePath = $path + $($dbname)

foreach ($ScriptThis in $Objects | where {!($_.IsSystemObject)}) {
#Need to Add Some mkDirs for the different $Fldr=$ScriptThis.GetType().Name
$scriptr = new-object ('Microsoft.SqlServer.Management.Smo.Scripter') ($SMOserver)
$scriptr.Options.AppendToFile = $True
$scriptr.Options.AllowSystemObjects = $False
$scriptr.Options.ClusteredIndexes = $True
$scriptr.Options.DriAll = $True
$scriptr.Options.ScriptDrops = $False
$scriptr.Options.IncludeHeaders = $True
$scriptr.Options.ToFileOnly = $True
$scriptr.Options.Indexes = $True
$scriptr.Options.Permissions = $True
$scriptr.Options.WithDependencies = $False

$ScriptDrop = new-object ('Microsoft.SqlServer.Management.Smo.Scripter') ($SMOserver)
$ScriptDrop.Options.AppendToFile = $True
$ScriptDrop.Options.AllowSystemObjects = $False
$ScriptDrop.Options.ClusteredIndexes = $True
$ScriptDrop.Options.DriAll = $True
$ScriptDrop.Options.ScriptDrops = $True
$ScriptDrop.Options.IncludeHeaders = $True
$ScriptDrop.Options.ToFileOnly = $True
$ScriptDrop.Options.Indexes = $True
$ScriptDrop.Options.WithDependencies = $False

#This section builds folder structures.#&gt;
$TypeFolder=$ScriptThis.GetType().Name
if ((Test-Path -Path "$SavePath\$TypeFolder") -eq "true")
        {"Scripting Out $TypeFolder $ScriptThis"}
    else {new-item -type directory -name "$TypeFolder"-path "$SavePath"}
$ScriptFile = $ScriptThis -replace "\[|\]"
$ScriptDrop.Options.FileName = "" + $($SavePath) + "\" + $($TypeFolder) + "\" + $($ScriptFile) + ".SQL"
$scriptr.Options.FileName = "$SavePath\$TypeFolder\$ScriptFile.SQL"

#This is where each object actually gets scripted one at a time.
$ScriptDrop.Script($ScriptThis)
$scriptr.Script($ScriptThis)
} #This ends the loop
} #This completes the function{% endhighlight %}
Great, we can script a database, now let's make it hassle free :) To accomplish this we will run the function from another script:
{% highlight powershell %}
# --- config ---
$cred = Get-Credential
$server = "localhost\sqlexpress"
$path = "D:\temp\db\"
$dbnames = @("AdventureWorks");

# --- body ---
# build powershell command
[System.IO.Directory]::SetCurrentDirectory($path)
$pscommand = ". " + $path + "\Script-DBObjectsIntoFolders.ps1;"
foreach($db in $dbnames)
{
  $pscommand += "Script-DBObjectsIntoFolders " + $server + " " + $db + " " + $path + ";"
}
$job = Start-Job -ScriptBlock {Invoke-Expression $args[0]} -Credential $cred -ArgumentList @($pscommand)
Wait-Job $job
Receive-Job $job
$date = get-date -format yyyyMMddHHmm
git add .
git commit -am "Database snapshot $date"{% endhighlight %}
So what exactly is happening here:

<em>config</em> section allows you to set the parameters for the db:
<ul>
<ul>
<ul>
	<li>$cred - credentials, which i strongly believe should not be stored in plain text - that's why we use Get-Credential</li>
	<li>$server - server + instance name of your server (in my case it will be my local named instance of SQL Express</li>
	<li>$path - the path where you want to store the repository</li>
	<li>$dbnames - array of database names in case you have more than one that you want to get at one time</li>
</ul>
</ul>
</ul>
{% highlight powershell %}[System.IO.Directory]::SetCurrentDirectory($path){% endhighlight %}
Sets the path for git and allows you to use .\ as $path if you run the script from the current directory (as opposite to a scheduled task for example)
{% highlight powershell %}$job = Start-Job -ScriptBlock {Invoke-Expression $args[0]} -Credential $cred -ArgumentList @($pscommand){% endhighlight %}
Runs the job in the background. The primary purpose of jobs is to be able to create multithread scripts but in our case we just want to be able to have some handle to the time consuming task so we can wait for it before executing git.
{% highlight powershell %}Wait-Job $job{% endhighlight %}
Is stopping the execution of the script until $job finishes.
{% highlight powershell %}Receive-Job $job{% endhighlight %}
Is displaying the results of the job. You should see here what was scripted from the database or errors in case of failure.

<strong>How you probably want your workflow to look like.</strong>

Assuming you are like me - you just want to have regular snapshots of the database - this is how to get started:
<ol>
	<li>Create a directory for your project</li>
	<li>Copy both scripts there (the first one should be called: Script-DBObjectsIntoFolders.ps1)</li>
	<li>Set $server, $path and $dbnames in the second script</li>
	<li>Run it! (you will get errors like "<em>git.cmd : fatal: Not a git repository (or any of the parent directories): .git</em>" but don't worry, everything is fine)</li>
	<li>Assuming that the scripting of the db was successful run:
{% highlight powershell %}
git init
git -commit "Initial commit"{% endhighlight %}
</li>
	<li> Whenever you would like to have a snapshot of the db objects just run the second script.</li>
</ol>
For your convenience both scripts are available for <a href="https://skydrive.live.com/redir.aspx?cid=3bf2002e065bc195&amp;resid=3BF2002E065BC195!304&amp;parid=3BF2002E065BC195!303" target="_blank">download</a> from my SkyDrive.
