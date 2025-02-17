# Powershell Snippets

Some handy code snippets for powershell :)

## String manipulation

**Base64 Decode/Encode**

- Decode: ```[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("dGVzdA=="))```
- Encode: ```[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("test"))```

**Create files based on files**

Create md file for each file found in current folder and remove first and last chars:

```powershell
foreach ($file in (dir | select-object name)){New-Item ($file.name.Substring(3, $file.name.Length-7)+".md") -ItemType file}
```

**Create a password**

Option 1

```powershell
Function New-Password([int] $length, $pw = "")
{    
    $rng = New-Object System.Random
    for($i=0;$i -lt $length;$i++) { $pw = $pw +[char]$rng.next(33,126) }
    return $pw
}
New-Password 15
```

Option 2

```powershell
$pwlength = 20 # Something between 8 and 32

# Create a password
Function New-RandomPassword{
    Param([ValidateRange(8, 32)] [int] $Length = 16)
    $AsciiCharsList = @()

    foreach ($a in (33..126)){ $AsciiCharsList += , [char][byte]$a }

    do {
        $Password = ""
        $loops = 1..$Length
        Foreach ($loop in $loops) { $Password += $AsciiCharsList | Get-Random }
    }
    until ($Password -match "(?=^.{8,32}$)((?=.*\d)(?=.*[A-Z])(?=.*[a-z])|(?=.*\d)(?=.*[^A-Za-z0-9])(?=.*[a-z])|(?=.*[^A-Za-z0-9])(?=.*[A-Z])(?=.*[a-z])|(?=.*\d)(?=.*[A-Z])(?=.*[^A-Za-z0-9]))^.*")
    return $Password
}

# Looks cool and why use the first best choice? lol
for ($i = 0; $i -lt ($pwlength); $i++) {
    $line = ""
    for ($j = 0; $j -lt (5); $j++) { $line += "$(New-RandomPassword($pwlength)) " }
    Write-Output($line) -ForegroundColor Green
}

# Result
Write-Output("Password:") -ForegroundColor black -BackgroundColor yellow -NoNewline; Write-Output(" " + $(New-RandomPassword($pwlength))) -ForegroundColor Red
```

Option 3

```powershell
$buffer = New-Object byte[] 32;
[System.Security.Cryptography.RandomNumberGenerator]::Create().GetBytes(($buffer));
return [BitConverter]::ToString($buffer).Replace("-", [string]::Empty);
```

## System information

**Last boot time**

```powershell
Write-Output "System boot:" (Get-CimInstance -ClassName win32_operatingsystem | Select-Object -ExpandProperty LastBootUpTime)
```

**Last Installation Date**

```powershell
Get-ChildItem -Path HKLM:\System\Setup\Source* | ForEach-Object {Get-ItemProperty -Path Registry::$_} | Select-Object ProductName, ReleaseID, CurrentBuild, @{n="Install Date"; e={([DateTime]'1/1/1970').AddSeconds($_.InstallDate)}} | Sort-Object "Install Date"
```

**Get WiFi Passwords**

Add more cultures if needed

```powershell
$keyword = @{"de-DE" = 'Schlüsselinhalt'; "en-US" = 'Key Content'}
Invoke-Expression -Command '(netsh wlan show profiles) | Select-String "\:(.+)$" | %{$name=$_.Matches.Groups[1].Value.Trim(); $_} | %{(netsh wlan show profile name="$name" key=clear)} | Select-String ($keyword[(get-culture).Name]+"\W+\:(.+)$") | %{$pass=$_.Matches.Groups[1].Value.Trim(); $_} | %{[PSCustomObject]@{ WiFi=$name;PASSWORD=$pass }} | Format-Table -AutoSize'
```

**Get WinSAT information**

```powershell
# Get WinSAT Data (XML)
WinSAT formal
$path = Get-ChildItem -Path 'C:\Windows\Performance\WinSAT\DataStore\*Formal.*.xml' | Sort-Object -Property CreationTime -Descending | Select-Object -First 1 -ExpandProperty FullName 

# Parse as XML
$xml = [xml]::new()
$xml.Load($Path)

# Read XML
$node = $xml.WinSAT.Metrics.CPUMetrics.CompressionMetric
'CPU Compression Performance is {0} {1}' -f $node.'#text', $node.units
'CPU Manufacturer is {0} ' -f $xml.WinSAT.SystemConfig.Processor.Instance.Signature.Manufacturer.friendly
```

**Windows Defender Stats**

```powershell
$DefenderStatus = (Get-Service WinDefend -ErrorAction SilentlyContinue).Status
if ($DefenderStatus -ne "Running") {
    throw "The Windows Defender service is not currently running"
}
Get-MpComputerStatus
```

**Get Win10 key**

```powershell
(Get-WmiObject -query ‘select * from SoftwareLicensingService’).OA3xOriginalProductKey
```

**Install apps**

Get install apps from app-store

```powershell
Get-AppxPackage | Select-Object -Property Name, Status, Version, InstallLocation | Format-Table
```

List all programs installed on Windows (and ignore the ones from Microsoft)

```powershell
Get-WMIObject -Query "SELECT * FROM Win32_Product Where Not Vendor Like '%Microsoft%'" | Format-Table
```

List files in programs folder:

```powershell
$Path = "$Env:ProgramData\Microsoft\Windows\Start Menu\Programs"
$StartMenu = Get-ChildItem $Path -Recurse -Include *.lnk

ForEach ($Item in $StartMenu) {
   $Shell = New-Object -ComObject WScript.Shell
   $Properties = @{
        ShortcutName = $Item.Name
        Target = $Shell.CreateShortcut($Item).targetpath
        }
    New-Object PSObject -Property $Properties
}

[Runtime.InteropServices.Marshal]::ReleaseComObject($Shell) | Out-Null
```

List installed Windows Store Apps (and ignore some):

```powershell
#Requires -RunAsAdministrator

Import-Module Appx
$Packages = Get-AppxPackage
$Whitelist = @('*WindowsCalculator*', '*MSPaint*', '*Office.OneNote*', '*Microsoft.net*', '*MicrosoftEdge*', '*Microsoft*') ## Ignore MS Stuff

## Remove all things to ignore
ForEach($App in $Packages){
    $Matched = $false
    Foreach($Item in $Whitelist){
        If($App -like $Item){
            $Matched = $true
            break
        }
    }

    if($matched -eq $false){
        [PSCustomObject]@{
        Name = $App.Name
        Location = $App.InstallLocation
        }
    }
}
```

List all Start Menu Programs and their paths

```powershell
ForEach ($Item in Get-ChildItem "$Env:ProgramData\Microsoft\Windows\Start Menu\Programs" -Recurse -Include *.lnk) {
    $Shell = New-Object -ComObject WScript.Shell
    $Properties = @{ ShortcutName = $Item.Name; Target = $Shell.CreateShortcut($Item).targetpath }
    New-Object PSObject -Property $Properties
}
```

## System Config

Enable Remote Desktop

```powershell
(Get-WmiObject Win32_TerminalServiceSetting -Namespace root\cimv2\TerminalServices).SetAllowTsConnections(1,1) | Out-Null
(Get-WmiObject -Class "Win32_TSGeneralSetting" -Namespace root\cimv2\TerminalServices -Filter "TerminalName='RDP-tcp'").SetUserAuthenticationRequired(0) | Out-Null
Get-NetFirewallRule -DisplayName "Remote Desktop*" | Set-NetFirewallRule -enabled true
```

Run Repair-Volume on all valid drives

```powershell
Get-Volume | Where { $_.OperationalStatus -eq "OK" -and $_.DriveType -ne "CD-ROM" -and $_.FileSystemType -ne "Unknown" -and $_.DriveLetter.length -ne 0} | Foreach-Object { Write-Host("Checking Drive: " + $_.DriveLetter); Repair-Volume -DriveLetter $_.DriveLetter }
```

## Take a screenshot

Take a screenshot and save the image on your desktop:

```powershell
Add-Type -AssemblyName System.Windows.Forms
Add-type -AssemblyName System.Drawing
$Screen = [System.Windows.Forms.SystemInformation]::VirtualScreen
$bitmap = New-Object System.Drawing.Bitmap $Screen.Width, $Screen.Height
$graphic = [System.Drawing.Graphics]::FromImage($bitmap)
$graphic.CopyFromScreen($Screen.Left, $Screen.Top, 0, 0, $bitmap.Size)
$bitmap.Save([Environment]::GetFolderPath("Desktop") + "\Screenshot.bmp")
```

## Sound

Tune the guitar

```powershell
82, 110, 146, 196, 246, 329 | Foreach-Object {[console]::beep($_,4000)} # E,A,D,G,B,E
```

## Speech

Here are two examples using System.Speech.Synthesis.SpeechSynthesizer to read out some given text with powershell (doesn't work with powershell 7):

```powershell
[Reflection.Assembly]::LoadWithPartialName('System.Speech') | Out-Null
$tts = New-Object System.Speech.Synthesis.SpeechSynthesizer
$tts.Speak("OMG I can speak!")
```

It is possible to add [SSML](https://www.w3.org/TR/speech-synthesis) to set pitch and language:

```powershell
Add-Type -AssemblyName System.speech
$tts = New-Object System.Speech.Synthesis.SpeechSynthesizer

$Phrase = '
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" 
    xml:lang="en-US">
    <voice xml:lang="en-US">
    <prosody rate="1">
        <p>Normal pitch. </p>
        <p><prosody pitch="x-high"> High Pitch. </prosody></p>
    </prosody>
    </voice>
</speak>
'
$tts.SpeakSsml($Phrase)
```

This version uses the SAPI.SpVoice COM object and works with powershell 5 and 7:

```powershell
$sp = New-Object -ComObject SAPI.SpVoice
$sp.Speak("Time for the $((Get-Date).DayOfWeek) shuffle")
```

## Web

Download Images

```powershell
$url = [System.Uri]"https://example.org"
$regex = '(http(s)?:\/\/)([^\s(["<,>/]*)(\/)[^\s[",><]*(.png|.jpg|.gif|.jpeg|.svg)(\?[^\s[",><]*)?'
$useragent = "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"

$FilePath = (([System.Environment]::GetFolderPath([System.Environment+SpecialFolder]::Desktop))+"\"+$url.Authority)
New-Item -ItemType Directory -Force -Path $FilePath | Select-Object | Out-Null

Add-Type -AssemblyName System.Web
$webClient = New-Object System.Net.WebClient
$webClient.Headers.Add("user-agent", $useragent)
$webClient.Headers.Add("Content-Type","application/x-www-form-urlencoded")
$webpage = $webclient.DownloadString($url)
$listImgUrls = $webpage.ToLower() | Select-String -pattern $regex -Allmatches | ForEach-Object {$_.Matches} | Select-Object $_.Value -Unique

foreach($imgUrlString in $listImgUrls) {
    try {
        [System.Uri]$imgUri = New-Object System.Uri -ArgumentList $imgUrlString
        $imgSaveDestination = $FilePath+"\"+([System.IO.Path]::GetFileName($imgUri.LocalPath))
        $webClient.DownloadFile($imgUri, $imgSaveDestination)       
        Write-Output("Downloaded '$imgUrlString' to '$imgSaveDestination'")
    }
    catch { Write-Host("Error: " + $imgUrlString + " - ") -ForegroundColor Yellow -NoNewline; Write-Host($_.Exception.Message) -ForegroundColor Red }
}
```

## Email and Calendar

Sind Mail with depreacted Send-MailMessage command

```powershell
$From = "You@gmail.com"
$To = "sombody@somewhere.com"
$Attachment = "D:\test.txt"
$Subject = "Subject"
$Body = "Hi"
$SMTPServer = "smtp.gmail.com"
$SMTPPort = "587"
Send-MailMessage -From $From -to $To -Subject $Subject -Body $Body -SmtpServer $SMTPServer -port $SMTPPort -UseSsl -Credential (Get-Credential) -Attachments $Attachment
```

Get a list of meetings occurring today

```powershell
$ns = New-Object -ComObject Outlook.Application.GetNamespace('MAPI')
$Start = (Get-Date).ToShortDateString()
$End = (Get-Date).ToShortDateString()
$appointments = $ns.GetDefaultFolder(9).Items
$appointments.IncludeRecurrences = $true
$appointments.Restrict("[MessageClass]='IPM.Appointment' AND [Start] > '$Start' AND [End] < '$End'") |  
ForEach-Object {
    if (-Not $_.IsRecurring ) { $_;  } else {
        try { $_.GetRecurrencePattern().GetOccurrence((Get-Date).ToString("yyyy-MM-dd") + " " + $_.Start.ToString("HH:mm")) } 
        catch {  }
		}
	} | Sort-Object -property Start | ForEach-Object { 
    $arrr = ($_.RequiredAttendees.split(';') | ForEach-Object { $_.Trim() } | ForEach-Object { $_.split(' ')[1] + ' ' + $_.split(' ')[0] } )
    $attendees = ($arrr -join " ").Replace(", ",",").TrimEnd(',')
    "`n`t`t[ ] $($_.Start.ToString("HH:mm")) - $($_.Subject.ToUpper()) with: $attendees"
}
```

## HyperV

**Install and config**

- Install Hyper V Feature: ```Install-WindowsFeature -Name Hyper-V -ComputerName localhost -IncludeManagementTools -Restart```
- Check Hyper V Installation: ```Get-WindowsFeature -Name Hyper-V -ComputerName HOSTNAME```
- Reconfigure the service ```sc config vmms start=auto```
- Stop and start the service ```sc stop vmms; sc start vmms```
- Remove ```VM Remove-VM -Name "Linux" -Force```

**Get VHD owner**

```powershell
param ($HyperVNodes,$VHDName)

Foreach ($HyperVNode in $HyperVNodes)
{
	$VMs = Get-WmiObject -Namespace "root\virtualization" -ComputerName $HyperVNode -class Msvm_ComputerSystem | where {$_.Caption -match "Virtual Machine"}
	Foreach ($VM in $VMs)
	{
		$VMSettingData = Get-WmiObject -Namespace "root\virtualization" -ComputerName $HyperVNode `
		-Query "Associators of {$VM} Where ResultClass=Msvm_VirtualSystemSettingData AssocClass=Msvm_SettingsDefineState" 
		$VirtualDiskResource = Get-WmiObject -Namespace "root\virtualization" -ComputerName $HyperVNode `
		-Query "Associators of {$VMSettingData} Where ResultClass=Msvm_ResourceAllocationSettingData AssocClass=Msvm_VirtualSystemSettingDataComponent" | `
		Where-Object { $_.ResourceSubType -match "Microsoft Virtual Hard Disk" }
		$VHDPath = $null
		Foreach ($VHD in $VirtualDiskResource)
		{
			$VHDPath = $VHDPath + $VHD.Connection[0] + ","
		}
		if ($VHDPath -Match $VHDName)
		{
			Write-Output "The VM named $($VM.ElementName) is connected to the VHD $VHDName"
		}
	}
}
```

## KMS

To set a KMS Server into Windows Managed Server’s configuration you need to execute the following command:

```powershell
slmgr /skms <KMS-FQDN>:1688
```

To trigger Windows activation you need to execute the following command:

```powershell
slmgr /ato
```

## IIS

IIS Passwords

```powershell
foreach ($site in Get-ChildItem IIS:\Sites) { "Site: " + $site.name + " - " + "User: " + $site.userName + "PW: " + $site.password }
```

Add website to pool and update binding

```powershell
Set-ItemProperty 'IIS:\\Sites\\$Site' ApplicationPool $AppPoolName  
Set-ItemProperty IIS:\\Sites\\$Site -Name bindings -Value (@{protocol="https";bindingInformation="\*:$Port:$Site"})

Start-WebSite -Name $site
Stop-WebSite -Name $site

Clear-ItemProperty IIS:\\Sites\\$Site -Name bindings
```

Get Binding Info

```powershell
[string]$BindingInfo = $Binding.Collection
[string]$IP = $BindingInfo.SubString($BindingInfo.IndexOf(" "),$BindingInfo.IndexOf(":")-$BindingInfo.IndexOf(" "))
[string]$Port = $BindingInfo.SubString($BindingInfo.IndexOf(":")+1,$BindingInfo.LastIndexOf(":")-$BindingInfo.IndexOf(":")-1)
```

## Random (sort this)

- Format new, raw disk: ```Get-Disk | Where-Object {$_.partitionstyle -eq 'raw' -and $_.OperationalStatus -like 'Online' } | Initialize-Disk -PartitionStyle MBR -PassThru | New-Partition -AssignDriveLetter -UseMaximumSize | Format-Volume -FileSystem NTFS -NewFileSystemLabel "newDisk" -Confirm:$false```
- Check if a specific port is open e.g. port 8080 ```netstat -ano | Select-String "8080"```
- Get all "svchost" process-ids: ```Get-Process "svchost" | select -expand id```
- View open connections for a given service: ```get-nettcpconnection | select local*,remote*,state,@{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Where-Object {$_.Process -eq "svchost"} | Format-Table```
- Check for Windows Updates: ```(New-Object -ComObject Microsoft.Update.AutoUpdate).DetectNow()```
- Export PowerShell command history to a file: ```Get-History | Export-CSV $env:USERPROFILE\Desktop\CommandHistory.CSV```
- Export all available powershell commands: ```Get-Command  | Export-CSV $env:USERPROFILE\Desktop\CommandsAvailable.CSV```
- Get last 10 installations: ```get-wmiobject Win32_ReliabilityRecords -computername 127.0.0.1 | select-object -first 10 Message | format-list *```
- Last 10 security event log entries: ```Get-EventLog Security -Newest 10```
- Get all help examples: ```Get-Command -CommandType cmdlet | % { (get-help $\_.name).examples }```
- Get Last Server Boot Time: ```([wmi]"").ConvertToDateTime((Get-WmiObject -Class Win32_OperatingSystem).LastBootuptime)```
- Search recursively for a certain string within files ```dir –r | select string "searchforthis"```
- Top5 processes using the most memory: ```ps | sort –property ws | select –last 5```
- Get all self-signed certs: ```Get-ChildItem -path cert:\\LocalMachine\\My```
- Create NIC Teaming: ```New-NetLBfoTeam –Name Guest –TeamMembers Guest-A,Guest-B -TeamingMode SwitchIndependent```
- Find something in (large) files: ```Get-Content myTestLog.log -wait | where { $\_ -match “WARNING” }```
- Read Registry Key: ```Get-ItemProperty -Path Registry::"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\" | format-list```
- Report all of the USB devices installed: ```gwmi Win32_USBControllerDevice -computername SERVER1 |fl Antecedent,Dependent```
- NTFS folder permissions for D:\Temp: ```Get-Acl "D:\Temp"```
- NTFS folder permissions for all files below D:\Temp: ```Get-ChildItem "D:\Temp" -recurse | Get-Acl```
- Get SHA1 hash sum of all files of the current folder: ```ForEach ($Item in Get-ChildItem $PWD -Include *.*) {Write-Host((Get-FileHash $Item.Name -Algorithm "SHA1").hash) $Item.Name }```

Add WebDAV to local path

```powershell
(Invoke-WebRequest https://webdav.domain.com/ -Method Options).Headers.DAV  
[String]$WebDAVShare = '\\webdav.domain.com@SSL/path/to/files'
New-PSDrive -Name S -PSProvider FileSystem -Root $WebDAVShare -Credential 'user@domain.tdl'
```

Set network location to Private for all networks:

```powershell
$networkListManager = [Activator]&#x3A;:CreateInstance([Type]&#x3A;:GetTypeFromCLSID([Guid]"{DCB00C01-570F-4A9B-8D69-199FDBA5723B}")) 
$connections = $networkListManager.GetNetworkConnections() 
$connections | % {$\_.GetNetwork().SetCategory(1)}
```
