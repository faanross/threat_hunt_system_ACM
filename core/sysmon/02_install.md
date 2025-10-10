
## **Installation and Setup**

### **Download**

Sysmon is available as part of the Sysinternals Suite:

**Official Microsoft Download**: https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon

**Direct Link**: https://download.sysinternals.com/files/Sysmon.zip


You can also download it via PowerShell:

```shell
# Download Sysmon (run as Admin)
Invoke-WebRequest -Uri https://download.sysinternals.com/files/Sysmon.zip -OutFile C:\temp\sysmon.zip

# Extract Sysmon archive
Expand-Archive C:\temp\sysmon.zip -DestinationPath C:\temp\sysmon
```


Always download `Sysmon` from the official Microsoft/Sysinternals sources to ensure you're getting an authentic, unmodified binary.


### Configuration File

Though one could technically just install Sysmon with its default configuration, that's really a huge missed opportunity. By simply running a few extra commands you can use the SwiftOnSecurity base config instead, which already represent a giant level up from the default config.

Then, as you become more comfortable with editing this config (which we'll explore later in this guide), you can refine and improve it over time.


**Open PowerShell as Admin and run:**

```powershell
# Download SwiftOnSecurity base config
Invoke-WebRequest -Uri https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml -OutFile C:\temp\sysmon\sysmon-config.xml
```



### Now We Can Install

In the same terminal run:
```powershell
# Install with the SwiftOnSecurity config
C:\temp\sysmon\Sysmon64.exe -accepteula -i C:\temp\sysmon\sysmon-config.xml
```

The `-accepteula` flag automatically accepts the license agreement. The `-i` flag performs the installation.

Note that Sysmon will validate the XML config file during installation, so if there are any breaking mistakes, it will notify you and not complete the installation.


### Confirm it's working

If the install succeeded, I still like running the following command just to make absolutely sure Sysmon is actually running.

```shell
Get-Service -Name "Sysmon64"
```


You should see the following output:
```powershell
Status   Name               DisplayName
------   ----               -----------
Running  Sysmon64           Sysmon64
```


You can also check to make sure that the Sysmon Event Log channel exists:
```powershell
Get-WinEvent -ListLog Microsoft-Windows-Sysmon/Operational
```


In this case the output should look something like:
```powershell
LogMode   MaximumSizeInBytes RecordCount LogName
-------   ------------------ ----------- -------
Circular            67108864       41859 Microsoft-Windows-Sysmon/Operational
```

This is also a good command to keep tabs on the amount of records, and space the logs are occupying on disk.



### Deploying Sysmon at Enterprise Scale

Deploying Sysmon across an entire enterprise environment can be a technically demanding and challenging undertaking. The complexity increases significantly when you're managing hundreds or thousands of endpoints across different locations, network segments, and organizational units.

While we can't go into exhaustive detail here about every consideration and edge case you might encounter, this overview will give you a basic understanding of the main deployment approaches available and help you determine which might work best for your organization.

#### Common Deployment Methods

When it comes to rolling out Sysmon to multiple systems simultaneously, you have several viable options to choose from, each with its own strengths depending on your existing infrastructure.

##### Group Policy Deployment

If your organization uses **Active Directory**, **Group Policy Objects (GPO)** provide a native Windows mechanism for deploying Sysmon across your domain. You can leverage GPO in a couple of ways: either through startup scripts that run when machines boot up, or through the built-in software deployment mechanisms that Windows provides. This approach works particularly well in traditional on-premises environments where Group Policy is already the primary management tool.

##### Configuration Management Tools

Many enterprises already use configuration management platforms to maintain consistency across their infrastructure. Tools like **Microsoft's System Center Configuration Manager (SCCM)**, **Ansible**, **Puppet**, or **Chef** can all be used to deploy **Sysmon** as part of your standard endpoint configuration baseline.

This approach is especially beneficial because it integrates Sysmon deployment into your existing change management and compliance workflows, making it easier to ensure every system has the proper monitoring in place.

##### Custom Deployment Scripts

For organizations that need more flexibility or don't have extensive configuration management infrastructure, **PowerShell** scripts offer a straightforward deployment method.

A typical deployment script would handle the entire installation process automatically:
- Downloading the Sysmon package from the official Microsoft source,
- Retrieving your custom configuration file from an internal repository,
- Extracting the files to the appropriate location, and
- Finally installing Sysmon with your specified configuration.

Here's a simple example of what such a script might look like:

```powershell
# Example deployment script
$SysmonUrl = "https://download.sysinternals.com/files/Sysmon.zip"
$ConfigUrl = "https://your-internal-repo/sysmonconfig.xml"
$InstallPath = "C:\Program Files\Sysmon"

# Download and extract
Invoke-WebRequest -Uri $SysmonUrl -OutFile "$env:TEMP\Sysmon.zip"
Expand-Archive -Path "$env:TEMP\Sysmon.zip" -DestinationPath $InstallPath -Force

# Download configuration
Invoke-WebRequest -Uri $ConfigUrl -OutFile "$InstallPath\sysmonconfig.xml"

# Install with configuration
& "$InstallPath\Sysmon64.exe" -accepteula -i "$InstallPath\sysmonconfig.xml"
```

This script automates the entire process, making it easy to run manually on individual systems or integrate into larger automation workflows.

##### Cloud-Based Management Platforms

For organizations that already leverage modern cloud management, particularly those with remote or mobile workforces, platforms like **Microsoft Intune** and other **Mobile Device Management (MDM)** solutions provide an excellent deployment path. These platforms allow you to push Sysmon to endpoints regardless of their physical location, as long as they maintain internet connectivity and are enrolled in your MDM system. This approach is increasingly popular as more organizations adopt hybrid or fully remote work models.

#### Choosing Your Approach

The best deployment method for your organization will depend on your existing infrastructure, management tools, and operational practices. Many enterprises actually use a combination of these approaches - perhaps Group Policy for traditional on-premises systems and Intune for remote workers. The key is ensuring consistent deployment and configuration across your entire environment, regardless of which technical method you use to achieve it.


### **Uninstallation (if needed)**

To remove Sysmon:

```cmd
Sysmon64.exe -u
```

This cleanly uninstalls the service and driver. Note that the Event Log channel and existing events will remain - you can manually delete the log file if desired.




