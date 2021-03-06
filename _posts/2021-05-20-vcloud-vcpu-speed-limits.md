---
layout: post
title: High Co-Stop on vCloud Director VMs
permalink: vcloud-vcpu-costop.html
description: Some Description
date: 2021-05-20 9:00:00
tags: vcloud hpc 
published: true
---
I've been running VM performance tests in vCloud Director, and noticed something strange when the VMs are at 100% CPU utilization even when they're the only VM on the ESXi host.  For example, I had a 48 vCPU VM running on a 48-core ESXi host.  At 100% CPU utilization, there was about 20% co-stop as reported by vROps.  

![CPU Usage and CoStop]({{ site.baseurl }}images\vcloud-vcpu-costop\cpu-costop.png){: .center-image }

On further investigation, I saw that CPU Usage was 111 GHz, but CPU Demand was 132 GHz.

![Usage MHz and Demand]({{ site.baseurl }}images\vcloud-vcpu-costop\usage-demand.png){: .center-image }

I thought it was odd that first of all, demand would be higher than usage when the VM was assigned all 48 cpus, and second of all that it wouldn't be grante that if it was available.  This line of thinking reminded me that Turbo Boost exists, and maybe the host is trying to run faster than the rated 2.39GHz per core, but the VMs can't.

vCloud Director has a setting at the Org VDC level to specify the vCPU speed.  By default, it is set to 1 GHz.  What this does is sets a CPU Limit on each VM in the Org VDC at whatever you specify the CPU speed to be.  Knowing this, I had already set the vCPU speed to match the physical clock speed of 2.39 GHz.  After encountering this issue, I decided to raise the vCPU speed to 5 GHz (faster even than Turbo Boost) to effectively remove the limit entirely.  Unfortunately, in vCloud Director it is not possible to set the VM limit to "Unlimited" as you would in vCenter directly.

![vCloud vCPU Speed]({{ site.baseurl }}images\vcloud-vcpu-costop\vcd-cpu-speed.png){: .center-image }

Once this was done, I used PowerCLI to raise the limits on the VM:
```posh
get-vm -name "CPUTestVM" | Get-VMResourceConfiguration | Set-VMResourceConfiguration -CpuLimitMhz "240000"
```
Once this was done, CPU Co-Stop was 0% when CPU Usage was 100%, and CPU Demand matched CPU Usage.

![CPU Metrics After]({{ site.baseurl }}images\vcloud-vcpu-costop\cpu-metrics-after.png){: .center-image }

So be sure to remember Turbo Boost when you're setting the vCPU speed in vCloud Director!
