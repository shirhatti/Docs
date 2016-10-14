---
uid: tutorials/nano-server
---
<a name=nano-server></a>

  # ASP.NET Core on Nano Server

By [Sourabh Shirhatti](https://twitter.com/sshirhatti)

Attention: This tutorial uses a pre-release version of the Nano Server installation option of Windows Server Technical Preview 5. You may use the software in the virtual hard disk image only to internally demonstrate and evaluate it. You may not use the software in a live operating environment. Please see [https://go.microsoft.com/fwlink/?LinkId=624232](https://go.microsoft.com/fwlink/?LinkId=624232) for specific information about the end date for the preview.

In this tutorial, you'll take an existing ASP.NET Core app and deploy it to a Nano Server instance running IIS.

  ## Introduction

Nano Server is an installation option in Windows Server 2016, offering a tiny footprint, better security and better servicing than Server Core or full Server. Please consult the official [Nano Server documentation](https://technet.microsoft.com/en-us/library/mt126167.aspx) for more details.  There are 3 ways for you try out Nano Server for yourself:

1. You can download the Windows Server 2016 Technical Preview 5 ISO file, and build a Nano Server image

2. Download the Nano Server developer VHD

3. Create a VM in Azure using the Nano Server image in the Azure Gallery. If you don’t have an Azure account, you can get a free 30-day trial

In this tutorial, we will be using the pre-built [Nano Server Developer VHD](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula)  from Windows Server Technical Preview 5.

Before proceeding with this tutorial, you will need the [published](../publishing/index.md) output of an existing ASP.NET Core application. Ensure your application is built to run in a **64-bit** process.

  ## Setting up the Nano Server Instance

[Create a new Virtual Machine using Hyper-V](https://technet.microsoft.com/en-us/library/hh846766.aspx) on your development machine using the previously downloaded VHD. The machine will require you to set an administrator password before logging on. At the VM console, press F11 to set the password before the first log in.

After setting the local password, you will manage Nano Server using PowerShell remoting.

  ### Connecting to your Nano Server Instance using PowerShell Remoting

Open an elevated PowerShell window to add your remote Nano Server instance to your `TrustedHosts` list.

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": ["code", "ps1"]} -->

````

   $nanoServerIpAddress = "10.83.181.14"
   Set-Item WSMan:\localhost\Client\TrustedHosts "$nanoServerIpAddress" -Concatenate -Force
   ````

`NOTE:` Replace the variable `$nanoServerIpAddress` with the correct IP address.

Once you have added your Nano Server instance to your `TrustedHosts`, you can connect to it using PowerShell remoting

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": ["code", "ps1"]} -->

````

   $nanoServerSession = New-PSSession -ComputerName $nanoServerIpAddress -Credential ~\Administrator
   Enter-PSSession $nanoServerSession
   ````

A successful connection results in a prompt with a format looking like: `[10.83.181.14]: PS C:\Users\Administrator\Documents>`

  ## Creating a file share

Create a file share on the Nano server so that the published application can be copied to it. Run the following commands in the remote session:

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": ["code", "ps1"]} -->

````

   mkdir C:\PublishedApps\AspNetCoreSampleForNano
   netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=yes
   net share AspNetCoreSampleForNano=c:\PublishedApps\AspNetCoreSampleForNano /GRANT:EVERYONE`,FULL
   ````

After running the above commands you should be able to access this share by visiting `\\<nanoserver-ip-address>\AspNetCoreSampleForNano` in the host machine's Windows Explorer.

  ## Open port in the Firewall

Run the following commands in the remote session to open up a port in the firewall to listen for TCP traffic.

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": ["code", "ps1"]} -->

````

   New-NetFirewallRule -Name "AspNet5 IIS" -DisplayName "Allow HTTP on TCP/8000" -Protocol TCP -LocalPort 8000 -Action Allow -Enabled True
   ````

  ## Installing IIS

Add the `NanoServerPackage` provider from the PowerShell gallery. Once the provider is installed and imported, you can install Windows packages.

Run the following commands in the PowerShell session that was created earlier:

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": ["code", "ps1"]} -->

````

   Install-PackageProvider NanoServerPackage
   Import-PackageProvider NanoServerPackage
   Install-NanoServerPackage -Name Microsoft-NanoServer-Storage-Package
   Install-NanoServerPackage -Name Microsoft-NanoServer-IIS-Package
   ````

Note: Installing *Microsoft-NanoServer-Storage-Package* requires a reboot. This is a temporary work around and won't be required in the future.

To quickly verify if IIS is setup correctly, you can visit the url `http://<nanoserver-ip-address>/` and should see a welcome page. When IIS is installed, by default a web site called `Default Web Site` listening on port 80 is created.

  ## Installing the ASP.NET Core Module (ANCM)

The ASP.NET Core Module is an IIS 7.5+ module which is responsible for process management of ASP.NET Core HTTP listeners and to proxy requests to processes that it manages. At the moment, the process to install the ASP.NET Core Module for IIS is manual. You will need to install the version of the [.NET Core Windows Server Hosting bundle](https://dot.net/) on a regular (not Nano) machine. After installing the bundle on a regular machine, you will need to copy the following files to the file share that we created earlier.

On a regular (not Nano) machine run the following copy commands:

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": ["code", "ps1"]} -->

````

   copy C:\windows\system32\inetsrv\aspnetcore.dll ``\\<nanoserver-ip-address>\AspNetCoreSampleForNano``
   copy C:\windows\system32\inetsrv\config\schema\aspnetcore_schema.xml ``\\<nanoserver-ip-address>\AspNetCoreSampleForNano``
   ````

On a Nano machine, you will need to copy the following files from the file share that we created earlier to the valid locations. So, run the following copy commands:

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": ["code", "ps1"]} -->

````

   copy C:\PublishedApps\AspNetCoreSampleForNano\aspnetcore.dll C:\windows\system32\inetsrv\
   copy C:\PublishedApps\AspNetCoreSampleForNano\aspnetcore_schema.xml C:\windows\system32\inetsrv\config\schema\
   ````

Run the following script in the remote session:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "source": "/Users/shirhatti/src/Docs/aspnet/tutorials/nano-server/enable-ancm.ps1"} -->

````

   # Backup existing applicationHost.config
   copy C:\Windows\System32\inetsrv\config\applicationHost.config C:\Windows\System32\inetsrv\config\applicationHost_BeforeInstallingANCM.config

   Import-Module IISAdministration

   # Initialize variables
   $aspNetCoreHandlerFilePath="C:\windows\system32\inetsrv\aspnetcore.dll"
   Reset-IISServerManager -confirm:$false
   $sm = Get-IISServerManager

   # Add AppSettings section 
   $sm.GetApplicationHostConfiguration().RootSectionGroup.Sections.Add("appSettings")

   # Set Allow for handlers section
   $appHostconfig = $sm.GetApplicationHostConfiguration()
   $section = $appHostconfig.GetSection("system.webServer/handlers")
   $section.OverrideMode="Allow"

   # Add aspNetCore section to system.webServer
   $sectionaspNetCore = $appHostConfig.RootSectionGroup.SectionGroups["system.webServer"].Sections.Add("aspNetCore")
   $sectionaspNetCore.OverrideModeDefault = "Allow"
   $sm.CommitChanges()

   # Configure globalModule
   Reset-IISServerManager -confirm:$false
   $globalModules = Get-IISConfigSection "system.webServer/globalModules" | Get-IISConfigCollection
   New-IISConfigCollectionElement $globalModules -ConfigAttribute @{"name"="AspNetCoreModule";"image"=$aspNetCoreHandlerFilePath}

   # Configure module
   $modules = Get-IISConfigSection "system.webServer/modules" | Get-IISConfigCollection
   New-IISConfigCollectionElement $modules -ConfigAttribute @{"name"="AspNetCoreModule"}

   # Backup existing applicationHost.config
   copy C:\Windows\System32\inetsrv\config\applicationHost.config C:\Windows\System32\inetsrv\config\applicationHost_AfterInstallingANCM.config


   ````

`NOTE:` Delete the files `aspnetcore.dll` and `aspnetcore_schema.xml` from the share after the above step.

  ## Installing .NET Core Framework

If you published a portable app, .NET Core must be installed on the target machine. Execute the following Powershell script in a remote Powershell session to install the .NET Framework on your Nano Server.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "powershell", "source": "/Users/shirhatti/src/Docs/aspnet/tutorials/nano-server/Download-Dotnet.ps1"} -->

````powershell

   $SourcePath = "https://go.microsoft.com/fwlink/?LinkID=809115"
   $DestinationPath = "C:\dotnet"

   $EditionId = (Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' -Name 'EditionID').EditionId

   if (($EditionId -eq "ServerStandardNano") -or
       ($EditionId -eq "ServerDataCenterNano") -or
       ($EditionId -eq "NanoServer") -or
       ($EditionId -eq "ServerTuva")) {

       $TempPath = [System.IO.Path]::GetTempFileName()
       if (($SourcePath -as [System.URI]).AbsoluteURI -ne $null)
       {
           $handler = New-Object System.Net.Http.HttpClientHandler
           $client = New-Object System.Net.Http.HttpClient($handler)
           $client.Timeout = New-Object System.TimeSpan(0, 30, 0)
           $cancelTokenSource = [System.Threading.CancellationTokenSource]::new()
           $responseMsg = $client.GetAsync([System.Uri]::new($SourcePath), $cancelTokenSource.Token)
           $responseMsg.Wait()
           if (!$responseMsg.IsCanceled)
           {
               $response = $responseMsg.Result
               if ($response.IsSuccessStatusCode)
               {
                   $downloadedFileStream = [System.IO.FileStream]::new($TempPath, [System.IO.FileMode]::Create, [System.IO.FileAccess]::Write)
                   $copyStreamOp = $response.Content.CopyToAsync($downloadedFileStream)
                   $copyStreamOp.Wait()
                   $downloadedFileStream.Close()
                   if ($copyStreamOp.Exception -ne $null)
                   {
                       throw $copyStreamOp.Exception
                   }
               }
           }
       }
       else
       {
           throw "Cannot copy from $SourcePath"
       }
       [System.IO.Compression.ZipFile]::ExtractToDirectory($TempPath, $DestinationPath)
       Remove-Item $TempPath
   }

   ````

  ## Publishing the application

Copy over the published output of your existing application to the file share.

You may need to make changes to your *web.config* to point to where you extracted `dotnet.exe`. Alternatively, you can add `dotnet.exe` to your path.

Example of how a web.config might look like if `dotnet.exe` was **not** on the path:

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": ["code", "xml"]} -->

````

   <?xml version="1.0" encoding="utf-8"?>
   <configuration>
     <system.webServer>
       <handlers>
         <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
       </handlers>
       <aspNetCore processPath="C:\dotnet\dotnet.exe" arguments=".\AspNetCoreSampleForNano.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" forwardWindowsAuthToken="true" />
     </system.webServer>
   </configuration>
   ````

Run the following commands in the remote session to create a new site in IIS for the published app. This script uses the `DefaultAppPool` for simplicity. For more considerations on running under an application pool, see [Application Pools](../hosting/apppool.md#apppool.md).

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": ["code", "powershell"]} -->

````

   Import-module IISAdministration
   New-IISSite -Name "AspNetCore" -PhysicalPath c:\PublishedApps\AspNetCoreSampleForNano -BindingInformation "*:8000:"
   ````

  ## Known issue running .NET Core CLI on Nano Server and Workaround

If you’re using Nano Server Technical Preview 5 with .NET Core CLI, you will need to copy all DLL files from `c:\windows\system32\forwarders` to `c:\Program Files\dotnet\shared\Microsoft.NETCore.App\1.0.0\` and your  .NET Core binaries directory `c:\dotnet` (in this example), due to a bug that has since been fixed in later releases.

If you use `dotnet publish`, make sure to copy all DLL files from `c:\windows\system32\forwarders` to your publish directory as well.

If your Nano Server Technical Preview 5 build is updated or serviced, please make sure to repeat this process, in case any of the DLLs have been updated as well.

  ## Running the Application

The published web app should be accessible in browser at `http://<nanoserver-ip-address>:8000`. If you have set up logging as described in [Log creation and redirection](../hosting/aspnet-core-module.md#log-redirection.md), you should be able to view your logs at *C:\PublishedApps\AspNetCoreSampleForNano\logs*.
