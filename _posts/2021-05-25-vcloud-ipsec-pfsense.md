---
layout: post
title: Configure IPSec VPN Between PFSense and vCloud 10.2.2
permalink: vcloud-pfsense-vpn.html
description: Some Description
date: 2021-05-22 9:00:00
tags: vcloud pfsense 
published: true
---
Configuring an IPSec VPN on vCloud Director is easier than ever.  Here's a quick run through of how to do it with a PFSense firewall.

In vCloud, navigate to Networking > Edge Gateways and select your edge gateway.

![Edge Gateway]({{ site.baseurl }}images\vcloud-pfsense-vpn\1-edge-gateway.png){: .center-image }

Then, go to IPSec VPN and click New

![New IPSec]({{ site.baseurl }}images\vcloud-pfsense-vpn\2-new-vpn.png){: .center-image }

Fill out the form with Name, Description, and create your own pre-shared key (simply make something up, or use https://www.pskgen.com/).

![Settings]({{ site.baseurl }}images\vcloud-pfsense-vpn\3-settings1.png){: .center-image }

The Local Endpoint IP Address is the IP address of your edge gateway.  You can view available IPs by clicking the little (i) icon if you forget what that IP is.  In my environment, this is a NAT IP, not the actual external IP.  

![Local IPs]({{ site.baseurl }}images\vcloud-pfsense-vpn\4-local-ip1.png){: .center-image }

Enter the Local IP and local subnet that you want the VPN to connect to.

![Local Subnet]({{ site.baseurl }}images\vcloud-pfsense-vpn\5-local-subnet.png){: .center-image }

Under Remote Endpoint, enter the IP of your PFSense firewall and the networks on the PFSense side of the VPN.

![Remote Network]({{ site.baseurl }}images\vcloud-pfsense-vpn\6-remote-ip.png){: .center-image }

Click Save and that's really all there is to the vCloud configuration.

Now, in PF Sense, go to VPN > IPSec

![PF IPSec]({{ site.baseurl }}images\vcloud-pfsense-vpn\7-vpn-ipsec.png){: .center-image }

Click Add P1.  In general, we will use the default settings for everything.

Under General Information > Remote Gateway, enter the **public IP** of your vCloud Edge Gateway (not the NAT IP, if your edge gateway is using a NAT.)

![PF Remote Gateway]({{ site.baseurl }}images\vcloud-pfsense-vpn\8-pf-remote-gw.png){: .center-image }

This is the trickiest part of the whole setup.  Under Phase 1 Proposal (Authentication), you will need to set "My Identifier" to the **PF Sense Public IP** and the "Peer Identifier" to the Edge Gateway IP as defined as the Local Endpoint in vCloud Director.  So if the Edge Gateway is using a NAT, put the NAT address here.  In both settings, choose "IP address" from the dropdown.  Enter the Pre-Shared Key in the appropriate box.

![PF Remote Gateway]({{ site.baseurl }}images\vcloud-pfsense-vpn\9-pf-peers.png){: .center-image }

Leave the remaining settings as default, scroll to the bottom and click Save.

Under your new Tunnel, click Show Phase 2 Entries, then Add P2.  This is where we define the local and remote subnets for the tunnel, and should match what we put in vCloud Director.

Under Local Network, choose Network from the drop down, then enter the PFSense network.  Under Remote Network, enter the vCloud network.

![PF Remote Gateway]({{ site.baseurl }}images\vcloud-pfsense-vpn\10-pf-networks.png){: .center-image }

Leave the rest default, scroll down and click Save.  

Click "Enable" on your new Tunnel. Then click Apply Changes.  

![PF Remote Gateway]({{ site.baseurl }}images\vcloud-pfsense-vpn\11-pf-enable.png){: .center-image }

Then go to Status > IPSec and click Connect VPN.

![PF Remote Gateway]({{ site.baseurl }}images\vcloud-pfsense-vpn\12-pf-connect.png){: .center-image }

And you should be good to go!

![PF Remote Gateway]({{ site.baseurl }}images\vcloud-pfsense-vpn\13-pf-connected.png){: .center-image }



