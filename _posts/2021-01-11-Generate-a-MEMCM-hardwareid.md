---
layout: post
title: Generate a MEMCM HardwareID
---

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

	function Get-MEMCMHardwareID {
		[OutputType([String])]
		$SystemEnclosureInformation = Get-CimInstance -Namespace "root/cimv2" -ClassName "Win32_SystemEnclosure"
		$SystemEnclosureSerialNumber = $SystemEnclosureInformation | Select-Object -ExpandProperty "SerialNumber"
		$SystemEnclosureSMBIOSAssetTag = $SystemEnclosureInformation | Select-Object -ExpandProperty "SMBIOSAssetTag"
		$BaseBoardSerialNumber = Get-CimInstance -Namespace "root/cimv2" -ClassName "Win32_BaseBoard" | Select-Object -ExpandProperty "SerialNumber"
		$BIOSSerialNumber = Get-CimInstance -Namespace "root/cimv2" -ClassName "Win32_BIOS" | Select-Object -ExpandProperty "SerialNumber"
		$SystemEnclosureChassisType = $SystemEnclosureInformation | Select-Object -ExpandProperty "ChassisTypes"

		if (Get-IsChassisTypeLaptop -ChassisType $SystemEnclosureChassisType) {
			$MacAddress = "<Not used on laptop>"
		}
		else {
			$MacAddress = Get-CimInstance -Namespace "root/cimv2" -Query "SELECT Index, MACAddress FROM Win32_NetworkAdapterConfiguration WHERE IPEnabled=True" | Select-Object -First 1 -ExpandProperty MacAddress
		}

		$HashBytes = Get-HashFromString -String ("{0}!{1}!{2}!{3}!{4}" -f $SystemEnclosureSerialNumber,$SystemEnclosureSMBIOSAssetTag,$BaseBoardSerialNumber,$BIOSSerialNumber,$MacAddress)
		$HashString = Get-HexAsString -Bytes $HashBytes

		"2:$HashString"
	}

### Dependencies
[Get-HashFromString](https://github.com/Robert-LTH/Powershell/blob/master/Get-HashFromString.ps1)

[Get-HexAsString](https://github.com/Robert-LTH/Powershell/blob/master/Get-HexAsString.ps1)

[Get-IsChassisTypeLaptop](https://github.com/Robert-LTH/Powershell/blob/master/Get-IsChassisTypeLaptop.ps1)
