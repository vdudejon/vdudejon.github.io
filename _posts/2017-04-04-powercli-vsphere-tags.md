---
layout: post
title: Configure vSphere Tags Across Multiple vCenters using PowerCLI
permalink: tags-multiple-vcs.html
description: Some Description
date: 2017-04-04 9:00:00
tags: powercli vrops tags groups
published: true
---

vSphere Tags are something I didn't use very often until I started using vROps.  In vROps, it's possible to create a Custom Group based on the vSphere tag which can really speed up the creation of those Custom Groups.  In a large environment, however, it can be a real chore to create and assign tags on lots of objects in multiple locations.  Using PowerCLI, we can easily create and assign tags across multiple vCenters.

### Connect to All Your vCenters
You might have a more sophisticated way of doing this already, but I have a text document with all my vCenter IPs and simply import them using `get-content` and then connect to all of them, like so:

```posh
$vcenters = Get-Content c:\vcenters.txt
$creds = Get-Credential
Connect-VIServer $vcenters -Credential $creds
```

I have this saved as a poweshell script, so when I need to connect I just run connect-vcenters.ps1 and off we go!

### Create Tag Category
Before you can create a tag, you have to create a tag category.  There's already an excellent write up on tags and PowerCLI on the [official VMware blog,](https://blogs.vmware.com/PowerCLI/2014/03/using-vsphere-tags-powercli.html) so I won't restate it.  This step is easy because it works across multiple vCenters, so all you've got to do is run it the same way you would against a single vCenter:

```posh
New-TagCategory –Name "vROps Custom Group" -Description "vROps Custom Group Membership" -Cardinality multiple
```

This line will provide a generic tag category you can use to define vROps Custom Group membership.  You'll notice I omit the switch `EntityType` so that we can use this category against any object in vCenter.  I also set Cardinality to `multiple` in case a single object needs to be a member of multiple Custom Groups.  

### Create Tag
Next you'll need to create a tag in the category you just created.  This is trickier, because the `New-Tag` commandlet does *not* work against multiple vCenters.  To account for this, I put it in a loop for each vCenter:

```posh
foreach ($vcenter in $vcenters) {New-Tag -name "GroupName" -Category "vROps Custom Group" -Server $vcenter}
```
    
You'll need to define your own group name, but now you've created a tag ready to use in all of your vCenters.

### Assign Tags
Here was the real fun part.  `New-TagAssignment` also doesn't work with multiple vCenters.  So if you want to assign a tag to a list of VMs, you need to know the vCenter name and pass it through with the `-Server` switch.  When I was figuring this out, I was trying to assign tags to a list of VMs which didn't include their vCenters.  To make a long story short, here's how you do it!

```posh
#Import your list of VMs, then Get-VM on all of them
$vmlist = get-content c:\vmlist.txt
$vms = Get-VM $vmlist
   
#Loop through each VM, assign the tag
foreach ($vm in $vms) { $vm | New-TagAssignment -tag "GroupName" -Server (([uri]$vm.ExtensionData.Client.ServiceUrl).Host)}
```
 
`([uri]$vm.ExtensionData.Client.ServiceUrl).Host` was the the real gold find for this task. `Get-VM` doesn't have an obvious property for the parent vCenter, but it exists in `(Get-VM).Client`.  You can see it here as a property of a VM variable in PowerGUI:

![get-vm-client]({{ site.baseurl }}images\2017-4-4-powercli-vsphere-tags1.png){: .center-image }

Full disclosure: I did not discover this on my own.  I received help on the PowerCLI channel of the VMware{code} Slack group, which I highly recommend that you join!

### Create Custom Group in vROps
Anyway, that's how you know what vCenter the VM belongs to, and how you can pass it into the `-Server` switch.  Once you've done, that you can go into vROps and create a custom group full of VMs based on their tag:

![vrops-tag]({{ site.baseurl }}images\2017-4-4-powercli-vsphere-tags2.png){: .center-image }

And that's all there is to it!  Now you have a Custom Group in vROps with membership defined by a vSphere Tag!
