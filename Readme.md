### Lability ###
<img align="right" alt="Lability logo" src="https://raw.githubusercontent.com/VirtualEngine/Lability/dev/Lability.png">

The __Lability__ module enables simple provisioning of Windows Hyper-V development and
testing environments. It uses a declarative document for machine configuration.
However, rather than defining configurations in an external custom domain-specific
language (DSL) document, __Lability__ extends existing PowerShell Desired
State Configuration (DSC) configuration .psd1 documents with metadata that can
be interpreted by the module.

By using this approach, it allows the use of a single confiugration document to
describe all properties for provisioning Windows-centric development and/or test
environments.

The __Lability__ module will parse the DSC configuration document and provision
Hyper-V virtual machines according to the metadata contained within. When invoked,
__Lability__ will parse a DSC configuration document and automagically
provision the following resources:
* Virtual machine disk images
 * Download required evaluation Operating System installation media
 * Expand Windows Image (WIM) image files into Sysprep'd virtual machine parent disks, including Server 2016 TP4 and Nano server
 * Apply required/recommended DSC Windows updates
* Virtual networks
 * Create internal and external Hyper-V switches
* Virtual machines
 * Connect to the correct virtual switches
 * Inject DSC resources from the host machine
 * Inject a dynamically created Unattend.xml file
 * Inject external ISO, EXE and ZIP resources 
 * Inject the virtual machine's DSC document
 * Invoke the Local Configuration Manager (LCM) after Sysprep
 
An example DSC configuration document might look the following. Note: this is a
standard DSC .psd1 configuration document, but has been extended with specific
properties which the __Lability__ module can interpret.

```powershell
@{
    AllNodes = @(
		@{
			NodeName = 'DC1';
            Lability_ProcessorCount = 2;
			Lability_SwitchName = 'CORPNET';
			Lability_Media = '2012R2_x64_Standard_EN_Eval';
		},
		@{
			NodeName = 'APP1';
            Lability_ProcessorCount = 1;
			Lability_SwitchName = 'CORPNET';
			Lability_Media = '2012R2_x64_Standard_EN_Eval';
		}	
	)
	NonNodeData = @{
        Lability = @{
            Network = @(
                @{ Name = 'CORPNET'; Type = 'Internal'; }
			)
		}
	}
}
```
When `Start-LabConfiguration` is invoked with the above configuration document, it
will:
* Create an internal Hyper-V virtual switch named 'CORPNET'
* Download required Server 2012 R2 Standard Edition evaluation media
 * Create a Sysprep'd Server 2012 R2 Standard Edition parent VHDX
 * Install required/recommended DSC hotfixes
* Provision two Hyper-V virtual machines called 'DC1' and 'APP1'
 * Each VM will be given 2GB RAM (configurable default)
 * 'DC1' will be assigned 2 virtual CPUs
 * 'APP1' will be assigned 1 virtual CPU
 * Attach each virtual machine to the 'CORPNET' virtual switch
 * Create differencing VHDX for each VM
 * Inject a dynamically created Unattend.xml file into the differencing VHDX

A brief introduction to the __VirtualEngineLab__ module presented at the European
PowerShell Summit 2015 can be found __[here](https://www.youtube.com/watch?v=jefhLaJsG3E "Man vs TestLab")__.

## Versions

### Unreleased

* Fixes BandwidthReservationMode bug where duplicate virtual switches are created.
* Moves examples into the \Examples directory.
* Removes trailing space causing Resources.psd1 here string to not appear correctly in certain editors (like VS Code).
* Adds missing EnvironmentPrefix/Suffix support and tests to Start-Lab, Stop-Lab, Checkpoint-Lab and Restore-Lab.

### v0.9.7

* Adds backup and restore of the Lability host's settings:
 * New Export-LabHostConfiguration and Import-LabHostConfiguration cmdlets added.
* Adds environment prefix and suffix tagging:
 * Adds NonNodeData\Lability\EnvironmentPrefix and EnvironmentSuffix directives.
 * VM display names and VHD(X) files are named accordingly.
* Adds Linux VM support:
 * Custom media "OperatingSystem" property added.
 * Defaults to "Windows".
 * If "Linux" if specified, no resources are injected into the VM.
* Minor updates/fixes:
 * Renames LabHostDefaults.ps1 to LabHostDefault.ps1.
 * Renames LabVMDefaults.ps1 to LabVMDefault.ps1.
 * Adds -SecureBoot to Set-LabVMDefault.
 * Adds Generation property to LabMedia to ensure that VHD files result in Generation 1 VMs.
 * Documentation updates.

### v0.9.6

* Updates bundled DSC resources from PSGallery:
 * xHyper-V updated to v3.3.0.0.
 * xPendingReboot updated to v0.2.0.0.
* Minor updates/fixes:
 * Fixes parameter errors in WindowsFeature\Get-TargetResource calls (#62).
 * Suppresses SCCM client warning output in Get-LabHostConfiguration.
 * Changes Get-LabHostConfiguration output to PSObjects to improve readability.

### v0.9.5

* Minor updates/fixes:
 * Removes existing DSC resources in VM before copying local DSC resources.
 * Fixes VM generation bug when custom partition style is defined.
 * Fixes bug creating VMs from custom VHD media.
 * Updates bootstrap to remove existing DSC configurations and restart WMI.

### v0.9.4

* Adds DestinationPath directive to custom resources:
 * Permits specifying an alternative relative location than the default \Resources directory.
 * Explicitly exports all functions to allow auto-loading of the module.
* Minor updates/fixes:
 * Fixes bug expanding environment variables.

### v0.9.2

* Minor updates/fixes:
 * Fixes image enumeration with custom VHD media.

### v0.9.1

* Adds support for existing virtual switches:
 * Switches no longer need to be defined in the NonNodeData\Lability\Network configuration document.
 * If not defined, an existing virtual switch is used.
 * If there is no local virtual switch, an internal one is automatically created.
* Adds mutliple MAC address support.

### v0.9.0

* Renames module to Lability (#42).
* Adds NonNodeData\Lability\DSCResource directive:
 * Permits bootstrapping DSC resources from both the PSGallery and directly from Github.
* Adds manual node configuration:
 * Adds Test-LabNodeConfiguration cmdlet to test a node's configuration.
 * Adds Invoke-LabNodeConfiguration to install Lability certificates and download required DSC resources.


[__Lability__ image/logo attribution credit](https://openclipart.org/image/300px/svg_to_png/22734/papapishu-Lab-icon-1.png)