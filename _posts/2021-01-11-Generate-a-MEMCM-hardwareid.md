---
layout: post
title: Generate a MEMCM HardwareID
---

# Generate a MEMCM HardwareID

ClientIDManagerStartup.log shows which parts are used to generate the ID. The entry in the log looks like this:

> Computed HardwareID=2:<br />
	Win32_SystemEnclosure.SerialNumber=<br />
	Win32_SystemEnclosure.SMBIOSAssetTag=<br />
	Win32_BaseBoard.SerialNumber=<br />
	Win32_BIOS.SerialNumber=<br />
	Win32_NetworkAdapterConfiguration.MACAddress=

SHA1 has a length of 40 character and the part after 2: is 40 characters long which makes it probable that it is a SHA1 hash.

### What the log doesn't tell you

There are three more things that the log doesn't say.
  - The macaddress is replaced with `<Not used on laptop`> if the computer is a laptop.
  - If the computer isn't a laptop the address used is the macaddress from the first IpEnabled network adapter.
  - An exclamation mark (!) is used as separator
  - The input string should be encoded as Unicode

### The script
<script src="http://gist-it.appspot.com/https://github.com/Robert-LTH/Powershell/blob/master/Get-MEMCMHardwareID.ps1" ></script>

### Get-HashFromString
<script src="http://gist-it.appspot.com/https://github.com/Robert-LTH/Powershell/blob/master/Get-HashFromString.ps1" ></script>

### Get-HexAsString
<script src="http://gist-it.appspot.com/https://github.com/Robert-LTH/Powershell/blob/master/Get-HexAsString.ps1" ></script>

### Get-IsChassisTypeLaptop
<script src="http://gist-it.appspot.com/https://github.com/Robert-LTH/Powershell/blob/master/Get-IsChassisTypeLaptop.ps1" ></script>
