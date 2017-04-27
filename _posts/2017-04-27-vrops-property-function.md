---
layout: post
title: PowerCLI Function for Adding Custom Properties to vROps Objects
permalink: vrops-property-function.html
description: Some Description
date: 2017-04-27 14:04:04 -05:00
tags: "powercli vrops properties"
---

In my previous post, I showed how to add specific details (IP and FQDN) to vCenter objects ion vROps as custom properties.  That's a handy thing, but it's a pretty limited use case.  To expand the scope a little, I wrote a function that will add any custom property and value to any vROps object.

What this will allow you to do, is once you import the function you just call it with the vROps object, the custom property name, and the custom property value.  It will then add the property/value to the object and then you're off.  For example 
```posh
Add-OMCustomProperty -objname "dbserver1.vdude.net" -propname "Custom Properties|Custom Groups" -propvalue "SQL"
```
This would add a property called "Customer Groups" under a category called "Custom Properties" on the object "dbserver1.vdude.net", then add a value of "SQL."  Then you could create a custom group based on that.  In a previous post I showed how to use vSphere tags to accomplish something similar.  Doing it this way places all the ownership on vROps rather than relying on the vSphere tags.  Here is the function (also available in my [powershell github folder](https://github.com/vdudejon/Powershell):

```posh
## v 1.0 April 27 2017
## https://github.com/vdudejon/Powershell/blob/master/Add-OMCustomProperty.psm1

function Add-OMCustomProperty {
	param($objname, $propname, $propvalue)
	
	# Time stamp
	[DateTime]$NowDate = (Get-date)
	[int64]$NowDateEpoc = Get-Date -Date $NowDate.ToUniversalTime() -UFormat %s
	$NowDateEpoc = $NowDateEpoc*1000

	# New objects for API call
	$contentprops = New-Object VMware.VimAutomation.vROps.Views.PropertyContents
	$contentprop = New-Object VMware.VimAutomation.vROps.Views.PropertyContent
	
	$contentprop.StatKey = $propname
	$contentprop.Values = $propvalue
	$contentprop.Timestamps = $NowDateEpoc

	# Add custom property objects to contentproperties object
	$contentprops.Propertycontent = @($contentprop)
	
	$obj = Get-OMResource -Name $objname
	
	# Add properties to resource
	$obj.ExtensionData.AddProperties($contentprops)
}
```

Just import that, and now you can add any arbitrary property to any vROps object!  To improve upon this, I want to make it possible to accept an object from `Get-OMResource` so that I can do `Get-OMResource -name "severname" |  Add-OMCustomProperty`.  