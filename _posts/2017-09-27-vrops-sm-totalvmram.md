---
layout: post
title: vROps Supermetric for Cluster Total VM RAM Provisioned
permalink: vrops-sm-totalvmram.html
description: Some Description
date: 2017-09-27 14:04:04 -05:00
tags: "vrops supermetrics"
---

Am I crazy or is there not a metric that shows how much total RAM has been assigned to a VM at the cluster level?  Like if I have a cluster with 10VMs at 10GB of RAM each, I want a metric that tells me that cluster has 100GB of RAM assigned.  I lokoed high and low and could not find such a metric.  Luckily, vROps supermetrics make it easy to build exactly that.

I won't bother with a tutorial on how to create supermetrics, because those are all over the place.  But to cut to the chase, the supermetric is `(sum($adaptertype=VMWARE, objecttype=VirtualMachine, attribute=mem|actual.capacity, depth=2}))/1048576` 

Depth=2 here because we're tracking the sum of all VM configured RAM 2 levels above at the cluster object.  I also divide by 1048576 to convert from KB to GB.  I don't see a way to specify units in a supermetric, but if that were possible it would be ideal.  For now, I've called the metric VMAssignedMemoryGB.

Next, you add the Cluster Compute Resource as the object type, then activate it in your policy, and you're all set! 
