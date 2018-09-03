WCMSC 1 & 2 Install [OSP 10]

Method of Procedure

Prepared for: Verizon Wireless

# Table Of Contents #
________________

### [Preface] (#preface-1)
  * [Confidentiality, Copyright, and Disclaimer] (#confidentiality-copyright-and-disclaimer)
  * [About This Document] (#about-this-document)
  * [Audience] (#audience)
  * [Additional Background and Related Documents] (#additional-background-and-related-documents)
  * [Terminology] (#terminology)
  * [Staffing] (#staffing)

### [Details] (#details-1)
  * [Detailed Description and What the Work Will Accomplish] (#detailed-description-and-what-the-work-will-accomplish)
  * [Equipment Being Worked On; Effect on Any Other Equipment] (#equipment-being-worked-on-effect-on-any-other-equipment)
    + Systems
    + Other Affected Systems
  * [Safety Requirements for Personnel, Equipment, Tools, and Hazardous Material] (#safety-requirements-for-personnel-equipment-tools-and-hazardous-material)
  * [Software Backup and Change Control Requirements] (#software-backup-and-change-control-requirements)
  * [Test Sets, Test, and Expected Results] (#test-sets-test-and-expected-results)
  * [Detailed Step-by-Step Procedure and Additional  Information] (#detailed-step-by-step-procedure-and-additional-information)
    + Prerequisites
      + Firmware
        + All HP equipment should be updated to the same SPP version, as directed by the Verizon Architect. There are known issues with outdated versions of HP firmware including Emulex and IPv6 support. There is no specific recommendation for the version from Red Hat, but all equipment should be at the same version to prevent any errors or failures on a subset of hosts due to firmware bugs.
    + Software
    + Out-of-band Management
    + Hostnames and IP Information

      + [WCMSC1 Networking Information] (#wcmsc1-networking-information)
      + [WCMSC2 Networking Information] (#wcmsc2-networking-information)

    + Other External Systems

### [Installation Steps] (#installation-steps-1)
  * [Phase 1 - Configuring OSP Director] (#phase-1-configuring-osp-director)
    + Install the base Red Hat Enterprise Linux Server for Director
    + Prepare the System for the OSP Undercloud
    + Install and Configure OSP Director (START HERE AFTER KICKSTART)
    + Register Nodes for the Overcloud
    + Retrieve the Templates from a Git Repository
    + [Verizon-specific] Run the image_mod.sh script
    + Configure Heat Templates for Deployment
  * [Phase 2 - Deploying the Overcloud] (#phase-2-deploying-the-overcloud)
    + Prepare Director for Overcloud Deployment
    + Run the Deploy Command
  * [Phase 3 - Configuring the Overcloud] (#phase-3-configuring-the-overcloud)
    + Create Provider Networks
    + Configure SR-IOV Networks
    + Configure SR-IOV NIC Agent
    + Configure Fencing

### [Appendix I - Troubleshooting Techniques] (#appendix-i-troubleshooting-techniques-1)
   * [Reversion - Rollback packages and reboot if necessary] (#reversion-rollback-packages-and-reboot-if-necessary)
   * [Troubleshooting a failed overcloud deploy] (#troubleshooting-a-failed-overcloud-deploy)
   * [Deleting a failed stack] (#deleting-a-failed-stack)
   * [Interrupting stack creation] (#interrupting-stack-creation)
   * [Stack Failure Due to Post Deployment Validation] (#stack-failure-due-to-post-deployment-validation)

### [Appendix II - Post-deployment Configurations] (#appendix-ii-post-deployment-configurations-1)
  * [Create accounts and tenants] (#create-accounts-and-tenants)
    + Create OpenStack Tenants
    + Create OpenStack Users

  * [Injecting files into the overcloud images] (#injecting-files-into-the-overcloud-images)
  * [Creating a host aggregate/availability zone] (#creating-a-host-aggregateavailability-zone)
  * [Create Aggregates and Modify Pinning Configuration] (#create-aggregates-and-modify-pinning-configuration)
  * [Update Ironic Nodes to Boot from Disk] (#update-ironic-nodes-to-boot-from-disk)
  * [Modifying policy.json to Include the network_admin Role] (#modifying-policyjson-to-include-the-network_admin-role)
  * [Registering the Overcloud Nodes with subscription-manager] (#registering-the-overcloud-nodes-with-subscription-manager)
  * [Configuring the Ironic Rules Model] (#configuring-the-ironic-rules-model)
  * [Update the Compute Packages] (#update-the-compute-packages)
  * [Update the Controller Packages] (#update-the-controller-packages)

# Preface
______


## Confidentiality, Copyright, and Disclaimer

This is a Customer-facing document between Red Hat, Inc. and Verizon ("Client"). Copyright © 2016 Red Hat, Inc. All Rights Reserved. No part of the work covered by the copyright herein may be reproduced or used in any form or by any means – graphic, electronic, or mechanical, including photocopying, recording, taping, or information storage and retrieval systems – without permission in writing from Red Hat except as is required to share this information as provided with the aforementioned confidential parties.This document is not a quote and does not include any binding commitments by Red Hat. 


If acceptable, a formal quote can be issued upon request, which will include the scope of work, cost, and any customer requirements as      necessary


## About This Document

This is a confidential document between Red Hat, Inc. and Verizon Wireless detailing how to build an environment running RHEL-OSP 7.

## Audience

This document is provided for use by internal Verizon personnel in charge of implementing and maintaining a Red Hat OpenStack private cloud for Verizon use.

## Additional Background and Related Documents

This document also references additional information that can be found on Red Hat's documentation site at 
https://access.redhat.com/knowledge/docs/ and specifically at https://access.redhat.com/site/documentation/en- 
US/Red_Hat_Enterprise_Linux_OpenStack_Platform/. Documents specific to products covered in this solution include the following:


  * Getting Started Guide
  * Installation and Configuration Guide


The following Reference Architecture as in-depth use case studies that may be useful and are available at https://access.redhat.com/site/articles/reference-architecture:


  * Deploying Red Hat Enterprise Linux OpenStack Platform 7 with RHEL-OSP director 7.3


In addition, the EMC VNX OpenStack Juno Cinder Driver Best Practices Guide http://www.emc.com/collateral/white-papers/h14268-vnx-openstack-bp-wp.pdf and driver readme https://github.com/emc-openstack/vnx-direct-driver/blob/master/README_ISCSI.md should be referenced when configuring the VNX Cinder integration.

## Terminology
The table below provides a glossary of the terms and acronyms used within this document. 

| Term | Definition |
| --- | --- | 
| RHEL | Red Hat Enterprise Linux |
| Director | Undercloud server for deploying OpenStack |
| Nova | Compute service |
| Keystone | Identity service |
| Glance | Image service |
| Cinder | Persistent block storage service |
| Neutron | OpenStack networking service |
| Heat | OpenStack orchestration service |
| Puppet | System configuration tool used for OpenStack deployment |
| Cloud Controller | Master node in OpenStack configuration, usually providing public Nova API, Keystone identity services, Glance image services and scheduling |	
| Compute Node | OpenStack node running virtual machines on a hypervisor |
| Network node | OpenStack node providing network management and connectivity for compute nodes |	
| OVS | Open vSwitch (Virtual Switch) |	
| Flat network | Network without VLANs |
| Provider network | Representation of a physical network in OpenStack configuration that makes it available for cloud tenants |	
## Staffing
The Client will make certain staff members are available to the Red Hat Consultant in order to facilitate completion of the task list. The following persons were identified by the Client to support the engagement:

| Role | Client Assignment | Contact Information |
| --- | --- | ---|
| Project Manager | David L. Harris, Ph.D Manager - Network Element Evolution Planning. | O: 925.952.6808 M: 925.407.6583 David.Harris6@VerizonWireless.com |
| Architect | Kirk Campbell | M: 908.285.4315 kirk.campbell@verizonwireless.com |
| Architect | Niranjan B Avula DMTS, HQ Network Planning | M: 817.320.6079 Niranjan.Avula@VerizonWireless.com |
| Onsite Contractor | Ravi Ganapa Onsite Contractor | M: 925.785.7284 ravi.ganapa@saturnb2b.com | 

# Details
________________
## Detailed Description and What the Work Will Accomplish


>This Method of Procedure explains how to install Red Hat Enterprise Linux OpenStack Platform version 7 for a Verizon Wireless cloud using the Red Hat Enterprise Linux OpenStack Platform Director. The diagram below shows the components of the deployment and their network connectivity.

## Equipment Being Worked On; Effect on Any Other Equipment

![Environment-Layout](/uploads/f617331774696516afdb812c77478118/Environment-Layout.png)

###  Systems  
  * 4 x HP ProLiant DL360p Gen9 Servers (1 director, 3 controllers)
  * 1 x HP BladeSystem c7000 Chassis
  * 32 x HP ProLiant BL460c G9 Blade Servers (compute nodes)

###  Other Affected Systems
  * Cisco Nexus 5K Switches
  * Cisco Nexus 22xx Fabric Extenders

## Safety Requirements for Personnel, Equipment, Tools, and Hazardous Material

>Standard grounding, electrocution hazards apply  
    
## Software Backup and Change Control Requirements

 >TBD       
 >All changes approved by VzW Architect  

## Test Sets, Test, and Expected Results
  1. Test storage by uploading a minimal image to glance
  2. Test glance by booting an instance from the image
  3. Boot a test VM
    1. Test cinder by creating a cinder volume
    2. Attach the volume to the instance
    3. Format it with a file system, and test reading and writing to that file system
  4. Boot a pair of VMs on the default network
    1. VMs ping each other, and an external host to ensure connectivity
    2. Delete VMs to ensure deletions function
  5. Boot a pair of VMs using SR-IOV.
    1. Repeat ping test and deletion test from previous step
  6. Test a heat stack-create using a minimal heat template

## Detailed Step-by-Step Procedure and Additional Information
### Prerequisites
#### Firmware
>All HP equipment should be updated to the same SPP version, as directed by the Verizon Architect. There are known issues with outdated versions of HP firmware including Emulex and IPv6 support. There is no specific recommendation for the version from Red Hat, but all equipment should be at the same version to prevent any errors or failures on a subset of hosts due to firmware bugs.

#### Software
>Red Hat Enterprise Linux Server 7.2 disc image and the following OpenStack 7 repositories should be provided to the director.
>
  *  rhel-7-server-rh-common-rpms
  *  rhel-7-server-optional-rpms
  *  rhel-7-server-rpms
  *  rhel-ha-for-rhel-7-server-rpms
  *  rhel-7-server-extras-rpms
  *  rhel-7-server-openstack-9-rpms
  *  rhel-7-server-openstack-9-director-rpms

>Currently these repositories are already provided via a Satellite 6 server. This is to be used as a reference in case disconnected repository maintenance is required.

>In addition, the EMC VNX NaviSphere command line tool is required for VNX Cinder integration. The tool is available at the VNX Product Tool download site: https://download.emc.com/downloads/DL30837_Navisphere-CLI-(Linux-x64)-7.33.8.1.19.rpm 

#### Out-of-band Management
>iLO IP addresses and credentials for all systems should be configured and provided to the director. The out-of-band management network must be reachable from the director server and all controllers. In addition, the operator must have a system capable of accessing the iLO network for the OpenStack servers using a web browser and secure shell (SSH) to run the installation.

#### Hostnames and IP Information
>All system and network information need to be provided to the operator, as well as all credentials that will be used in the document. Tables for this information are provided below. All routing and VLAN configuration should be in place before starting. Routing should include connectivity from the director and controller external interfaces to the out-of-band management network. Controllers should have connectivity to EMC VNX SAN management for volume management.

## WCMSC1 Networking Information

| Subnet | Layer 3 segment | Network address | 192.168.70.192 | 27 | 2801 | fd00:4888:2000:f001:: | 64 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| address Gateway | VRRP address (Nexus 7706) | Gateway VRRP | 192.168.86.193 | 27 | 2801 | fd00:4888:2000:f001:524:0023::0 | 64 |
| address Net-device | 	Cisco Nexus 7706 | Gateway router 1 | 192.168.86.194 | 27 | 2801 | fd00:4888:2000:f001:524:0023::1 | 64 |
| address Net-device | Cisco Nexus 7706 | Gateway router 2 | 192.168.86.195 | 27 | 2801 | fd00:4888:2000:f001:524:0023::2 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller1 | 192.168.86.196 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::0 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller2 | 192.168.86.197 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::1 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller3 | 192.168.86.198 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::2 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller4 OSPd | 192.168.86.199 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::3 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller5 | 192.168.86.200 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::4 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller6 | 192.168.86.201 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::5 | 64 |
| address OSP haproxy VIP | Controller VIP (haproxy) | API External haproxy VIP | 192.168.86.202 | 27 | 	2801 | fd00:4888:2000:f001:524:0ff2::6 | 64 |
| Storage management | EMC VNX5400 control station | VNX CS0 | 192.168.86.203 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::7 | 64 |
| Storage management | EMC VNX5400 control station | VNX CS1 | 192.168.86.204 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::8 | 64 |
| Storage management | 	EMC VNX5400 storage processor | VNX SPA | 192.168.86.205 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::9 | 64 |
| Storage management | EMC VNX5400 storage processor | VNX SPB | 192.168.86.206 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::a | 64 |
| OSP neutron plugin | Tail-f NCS ML2 plugin | Tail-F NCS instance1 | 192.168.86.207 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::b | 64 |
| OSP neutron plugin | Tail-f NCS ML2 plugin | Tail-F NCS instance2 | 192.168.86.208 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::c | 64 |
| OSP neutron plugin VIP | Tail-f NCS ML2 plugin | Tail-f NCS ML2 VIP | 192.168.86.209 | 27 | 2801 | fd00:4888:2000:f001:524:0ff2::d | 64 |

## WCMSC2 Networking Information

| Subnet | Layer 3 segment | Network address | 192.168.70.192 | 27 | 2801 | fd00:4888:2000:f401:: | 64 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| address Gateway | VRRP address (Nexus 7706) | Gateway VRRP | 192.168.88.193 | 27 | 2801 | fd00:4888:2000:f401:524:0023::0 | 64 |
| address Net-device | Cisco Nexus 7706 | Gateway router 1 | 192.168.88.194 | 27 | 2801 | fd00:4888:2000:f401:524:0023::1 | 64 |
| address Net-device | Cisco Nexus 7706 | Gateway router 2 | 192.168.88.195 | 27 | 2801 | fd00:4888:2000:f401:524:0023::2 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller1 | 192.168.88.196 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::0 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller2 | 192.168.88.197 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::1 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller3 | 192.168.88.198 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::2 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller4 OSPd | 192.168.88.199 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::3 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller5 | 192.168.88.200 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::4 | 64 |
| address OSP controller | HP DL360 Gen9 | Controller6 | 192.168.88.201 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::5 | 64 |
| address OSP haproxy VIP | Controller VIP (haproxy) | API External haproxy VIP | 192.168.88.202 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::6 | 64 |
| Storage management | EMC VNX5400 control station | VNX CS0 | 192.168.88.203 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::7 | 64 |
| Storage management | EMC VNX5400 control station | VNX CS1 | 192.168.88.204 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::8 | 64 |
| Storage management | EMC VNX5400 storage processor | VNX SPA | 192.168.88.205 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::9 | 64 |
| Storage management | EMC VNX5400 storage processor | VNX SPB | 192.168.88.206 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::a | 64 |
| OSP neutron plugin | Tail-f NCS ML2 plugin | Tail-F NCS instance1 | 192.168.88.207 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::b | 64 |
| OSP neutron plugin | Tail-f NCS ML2 plugin | Tail-F NCS instance2 | 192.168.88.208 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::c | 64 |
| OSP neutron plugin VIP | Tail-f NCS ML2 plugin | Tail-f NCS ML2 VIP | 192.168.88.209 | 27 | 2801 | fd00:4888:2000:f401:524:0ff2::d | 64 |
	
## Other External Systems
  * Storage, including NFS for Cinder and Glance, should be configured prior to deployment.
  * At least one NTP server must be accessible by all nodes before deployment.
  * If DNS is used in the environment, servers should be accessible.


# Installation Steps
## Phase 1 - Configuring OSP Director 
### Install the base Red Hat Enterprise Linux Server for Director


  1. Access the iLO interface of the OSP Director server using a web browser, enter the iLO IP address for the Director from the equipment tables and log in using the provided credentials

	![credentials](/uploads/8c8380e812911282163023d256b88858/credentials.png)

  2. Open the integrated remote console by clicking the .NET or Java link depending on whether your platform is Windows or non-Windows, respectively. If prompted, agree to any security warnings


  3. In the remote console window, mount the ISO media by clicking Virtual Drives and selecting the Image File CD/DVD-ROM check box. Then select the .iso file on your local system 


  4. In the iLO page, expand Virtual Media and select Boot Order. Under one-time boot status, select CD/DVD Drive and click Apply

	![ilo](/uploads/63079b0c526908f353214e029ba30a4e/ilo.png)

  5. In the remote console window, power on the system by selecting Power Switch and Reset


  6. The ks.cfg file below is specifically for the wcmsc1 deployment.
		
		```
		# System authorization information
		auth --enableshadow --passalgo=sha512
		
		# Use CDROM installation media
		cdrom
		install
		poweroff
		
		# Use text install
		text
		
		# Run the Setup Agent on first boot
		firstboot --enable
		ignoredisk --only-use=sda
		
		# Keyboard layouts
		keyboard --vckeymap=us --xlayouts='us'
		
		# System language
		lang en_US.UTF-8
		
		# Network information
		network --bootproto=dhcp --device=eno1 --onboot=off --ipv6=auto
		network --bootproto=dhcp --device=eno2 --onboot=off --ipv6=auto
		network --bootproto=dhcp --device=eno3 --onboot=off --ipv6=auto
		network --bootproto=dhcp --device=eno49 --onboot=off --ipv6=auto
		network --bootproto=dhcp --device=eno4 --onboot=off --ipv6=auto
		network --bootproto=dhcp --device=eno50 --onboot=off --ipv6=auto
		network --bootproto=dhcp --device=ens2f0 --onboot=off --ipv6=auto
		network --bootproto=dhcp --device=ens2f1 --onboot=off --ipv6=auto
		network --hostname=wcmsc1-l-rh-ucld-01.hqplan.lab
		
		# Root password
		rootpw Root1234
		
		# System services
		services --enabled=sshd
		selinux --permissive
		firewall --disabled
		skipx
		
		# System timezone
		timezone America/Los_Angeles --isUtc --nontp
		
		# System bootloader configuration
		bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
		
		# Partition clearing information
		zerombr
		clearpart --all --initlabel 
		
		# Disk partitioning information
		part biosboot --fstype=biosboot --size=1
		part /boot --fstype="xfs" --ondisk=sda --size=500
		part pv.154 --fstype="lvmpv" --ondisk=sda --size=1 --grow
		volgroup rhel --pesize=4096 pv.154
		logvol / --fstype="xfs" --size=1 --name=root --vgname=rhel --grow
		logvol swap --fstype="swap" --size=4096 --name=swap --vgname=rhel
		
		%packages
		@^minimal
		@core
		kexec-tools
		ntp
		net-tools
		traceroute
		tcpdump
		nmap
		wget
		sysstat
		bind-utils
		sg3_utils
		curl
		screen
		redhat-lsb
		tmux
		nano
		pexpect
		strace
		numactl
		
		%end
		
		%addon com_redhat_kdump --enable --reserve-mb='auto'
		
		%end
		
		%post --log=/root/post-install.log
		set -x
		exec < /dev/tty3 > /dev/tty3
		chvt 3
		echo
		echo "################################"
		echo "# Running Post Configuration  #"
		echo "################################"
		
		## Build date used for motd and product file
		BUILDDATE=`date +%Y%m%d`
		NAME="RedHat 7.2"
		echo ${BUILDDATE} ${NAME} > /root/.foobar
		
		# Create login banner
		echo "Creating /etc/profile.d/info.sh"
		cat << INFO > /etc/profile.d/info.sh
		echo -e "
		
		\033[0;35m++++++++++++++++++++++++++++++++: \033[0;37mSystem Data\033[0;35m :+   ++++++++++++++++++++++++++++++++     
		\033[0;37mHostname \033[0;35m= \033[1;32m\`hostname\` 
		\033[0;35m+	\033[0;37mIPv4 Address \033[0;35m= \033[1;32m\`ip -o -4 addr show | sed 's/.* inet \([^/]*\).*/\1/' | sed -e :a -e '/$/N; s/\n/ /; ta'\`
		\033[0;35m+   \033[0;37mIPv6 Address \033[0;35m= \033[1;32m\`ip -o -6 addr show | sed 's/.* inet6 \([^/]*\).*/\1/' | sed -e :a -e '/$/N; s/\n/ /; ta'\`
		\033[0;35m+             \033[0;37mOS \033[0;35m= \033[1;32m\`lsb_release -d | awk '{print substr(\$0, index(\$0,\$2))}'\`
		\033[0;35m+         \033[0;37mKernel \033[0;35m= \033[1;32m\`uname -r\`
		\033[0;35m+         \033[0;37mUptime \033[0;35m= \033[1;32m\`uptime | sed 's/.*up ([^,]*), .*/1/' | xargs\`
		\033[0;35m+          \033[0;37mModel \033[0;35m= \033[1;32m\`lscpu | grep 'Model name:' | awk -F: '{print \$2}' | xargs\`
		\033[0;35m+            \033[0;37mCPU \033[0;35m= \033[1;32m\`lscpu | grep '^CPU(s):' | awk '{print \$2}'\`
		\033[0;35m+         \033[0;37mMemory \033[0;35m= \033[1;32m\`cat /proc/meminfo | grep MemTotal | awk {'print \$2'}\` kB
		\033[0;35m+     \033[0;37mHypervisor \033[0;35m= \033[1;32m\`lscpu | grep 'Hypervisor' | awk -F: '{print \$2}' | xargs\`
		\033[0;35m+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
		"
		echo -e "\033[0;m"
		INFO
		chmod 755 /etc/profile.d/info.sh
		
		#
		# configure IP addresses
		#
		
		echo "NETWORKING_IPV6=yes" > /etc/sysconfig/network
		echo "IPV6_AUTOCONF=no" >> /etc/sysconfig/network
		
		#
		# fix and don't remove ifcfg-lo
		#
		rm -rf /etc/sysconfig/network-scripts/ifcfg-*
		
		cat << ENS2F0 > /etc/sysconfig/network-scripts/ifcfg-ens2f0
		TYPE=Ethernet
		BOOTPROTO=none
		DEFROUTE=no
		PEERDNS=no
		PEERROUTES=no
		IPV4_FAILURE_FATAL=no
		IPV6INIT=yes
		IPV6_AUTOCONF=yes
		IPV6_DEFROUTE=yes
		IPV6_PEERDNS=yes
		IPV6_PEERROUTES=yes
		IPV6_FAILURE_FATAL=no
		NAME=ens2f0
		DEVICE=ens2f0
		ONBOOT=yes
		MASTER=bond0
		SLAVE=yes
		NM_CONTROLLED=no
		ENS2F0
		
		cat << ENS2F1 > /etc/sysconfig/network-scripts/ifcfg-ens2f1
		TYPE=Ethernet
		BOOTPROTO=none
		DEFROUTE=no
		PEERDNS=no
		PEERROUTES=no
		IPV4_FAILURE_FATAL=no
		IPV6INIT=yes
		IPV6_AUTOCONF=yes
		IPV6_DEFROUTE=yes
		IPV6_PEERDNS=yes
		IPV6_PEERROUTES=yes
		IPV6_FAILURE_FATAL=no
		NAME=ens2f1
		DEVICE=ens2f1
		ONBOOT=yes
		MASTER=bond0
		SLAVE=yes
		NM_CONTROLLED=no
		ENS2F1
		
		cat << BOND0 > /etc/sysconfig/network-scripts/ifcfg-bond0
		DEVICE=bond0
		BONDING_MASTER=yes
		BONDING_OPTS="mode=active-backup miimon=100"
		Type=Bond
		ONBOOT=yes
		NM_CONTROLLED=no
		PEERROUTES=no
		PEERDNS=no
		DEFROUTE=no
		BOND0
		
		cat << BOND0.2800 > /etc/sysconfig/network-scripts/ifcfg-bond0.2800
		TYPE="Ethernet"
		BOOTPROTO="static"
		NAME="bond0.2800"
		DEVICE="bond0.2800"
		ONBOOT="yes"
		NM_CONTROLLED="no"
		DEFROUTE="yes"
		IPV4_FAILURE_FATAL="no"
		IPV6INIT="no"
		IPV6_DEFROUTE="no"
		IPV6_FAILURE_FATAL="no"
		IPADDR="192.168.0.10"
		PREFIX="24"
		IPV6_PRIVACY="no"
		VLAN="yes"
		BOND0.2800
		
		cat << BOND0.2801 > /etc/sysconfig/network-scripts/ifcfg-bond0.2801
		TYPE="Ethernet"
		BOOTPROTO="static"
		NAME="bond0.2801"
		DEVICE="bond0.2801"
		ONBOOT="yes"
		NM_CONTROLLED="no"
		DEFROUTE="yes"
		IPV4_FAILURE_FATAL="no"
		IPV6INIT="yes"
		IPV6_DEFROUTE="yes"
		IPV6_FAILURE_FATAL="no"
		IPV6ADDR="fd00:4888:2000:f801:524:ff2::3/64"
		IPV6_DEFAULTGW="fd00:4888:2000:f801:524:23::"
		IPADDR="192.168.86.199"
		PREFIX="27"
		GATEWAY="192.168.86.193"
		IPV6_PRIVACY="no"
		VLAN="yes"
		BOND0.2801
  
		cat << BOND0.2804 > /etc/sysconfig/network-scripts/ifcfg-bond0.2800
		TYPE="Ethernet"
		BOOTPROTO="static"
		NAME="bond0.2804"
		DEVICE="bond0.2804"
		ONBOOT="yes"
		NM_CONTROLLED="no"
		DEFROUTE="yes"
		IPV4_FAILURE_FATAL="no"
		IPV6INIT="no"
		IPV6_DEFROUTE="no"
		IPV6_FAILURE_FATAL="no"
		IPADDR="192.168.1.4"
		PREFIX="24"
		IPV6_PRIVACY="no"
		VLAN="yes"
		BOND0.2804
		
		#
		# setup /etc/resolv.conf
		#
		echo "options timeout:2" > /etc/resolv.conf
		echo "search hqplan.lab" > /etc/resolv.conf
		echo "nameserver fd00:4888:2000:1::25" >> /etc/resolv.conf
		echo "nameserver fd00:4888:2000:1::24" >> /etc/resolv.conf
		
		#
		# fix ssh settings
		#
		sed -i.bak 's/GSSAPIAuthentication yes/GSSAPIAuthentication no/g' /etc/ssh/sshd_config
		sed -i.bak 's/#UseDNS yes/UseDNS no/g' /etc/ssh/sshd_config
		
		#
		# setup ntp
		#
		sed -i 's/^server/#server/g' /etc/ntp.conf
		sed -i '/server 3.rhel.pool.ntp.org iburst/a server fd00:4888:2000:1::26 iburst' /etc/ntp.conf
		systemctl start ntpd
		systemctl enable ntpd
		iptables -A INPUT -p udp --dport 123 -j ACCEPT
		iptables-save > /etc/sysconfig/iptables
		
		#
		# setup /etc/hosts
		#
		echo "" >> /etc/hosts
		echo "fd00:4888:2000:f801:524:ff2::3        wcmsc1-l-rh-ucld-01.hqplan.lab       wcmsc1-l-rh-ucld-01" >> /etc/hosts
		echo "192.168.86.199                        wcmsc1-l-rh-ucld-01.hqplan.lab        wcmsc1-l-rh-ucld-01" >> /etc/hosts
		
		#
		# OSPd items
		#
		useradd stack
		echo 'Root1234' | passwd stack --stdin
		echo "stack ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/stack
		chmod 0440 /etc/sudoers.d/stack
		
		sed i.bak `s/localhost /$(hostname -f) $(hostname -s) localhost/g` /etc/hosts
		
		#
		# disable NetworkManager
		#
		systemctl disable NetworkManager
		
		#
		# setup satellite connection
		#
		echo "/root/runonce&" >> /etc/rc.d/rc.local
		chmod 755 /etc/rc.d/rc.local
		
		cat << EOF > /root/runonce
		# wait for network to start
		sleep 300
		cd /root
		echo "fd00:4888:2000:1::26                cawcrhsd1v.hqplan.lab        cawcrhsd1v" >> /etc/hosts
		curl -O "http://\[fd00:4888:2000:1::26\]/pub/katello-ca-consumer-latest.noarch.rpm"
		rpm -ihv katello-ca-consumer-latest.noarch.rpm 
		subscription-manager register --org='Default_Organization' --activationkey='msc1' --force
		subscription-manager repos --enable rhel-7-server-openstack-9-rpms --enable rhel-7-server-openstack-9-director-rpms
		yum upgrade -y
		sed -i 's/runonce/d' /etc/rc.d/rc.local
		chmod 644 /etc/rc.d/rc.local
		chmod 644 /root/runonce
		reboot
		EOF
		chmod 755 /root/runonce
		
		#
		# clean up history file
		#
		history -c
		
		%end
		```

  7. When booting the installation disk, you will be greeted with the following window. Select the option Install Red Hat Enterprise Linux 7.2 but do not hit enter yet.

	![grub-menu](/uploads/97b277455467fbe14cfbdfbe252a8968/grub-menu.png)

  8. Hit tab. This will bring up the kernel command line. Append the following to the end of the kernel command line to enable the serial console. Replace <kickstart URL> with the location of the kickstart file to be used during installation.
	
	```
	ks=<kickstart URL> video=640x400 nomodeset console=ttyS1
	```	

	![grub-menu-2](/uploads/ae2b6a09677d1255b39f42914f335a8c/grub-menu-2.png)

  9. The kickstart should take care of the installation without any further user interaction.

  10. When the installation completes, the system should automatically reboot 

### Install and Configure OSP Director (START HERE AFTER KICKSTART)

Log into the Director server using an ssh client using stack@<director-external-ip> or the iLO remote console as described above. Login using the stack password defined when creating the account.

  1. Enable Repo

	```
	[stack@director ~]$ sudo subscription-manager repos --enable rhel-7-server-openstack-10-rpms
	```

  1. Install the packages required to install the undercloud

	```
	[stack@director ~]$ sudo yum install -y python-tripleoclient
	```

  1. Configure the Director Template. Copy over the undercloud.conf.sample file.

	```
	[stack@director ~]$ mkdir -p templates/undercloud-certs
	[stack@director ~]$ mkdir images
	[stack@director ~]$ cd /home/stack/templates
	[stack@director ~]$ cp /usr/share/instack-undercloud/undercloud.conf.sample ./undercloud.conf
	```

  1. Use crudini to modify undercloud.conf

	```
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT image_path /home/stack/images
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT local_ip 192.168.4.1/24
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT network_gateway 192.168.4.1
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT undercloud_public_vip 192.168.4.2
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT undercloud_admin_vip 192.168.4.3
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT undercloud_service_certificate /etc/pki/instack-certs/undercloud.pem
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT generate_service_certificate false
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT certificate_generation_ca local
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT local_interface eno1
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT network_cidr 192.168.4.0/24
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT masquerade_network 192.168.4.0/24
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT dhcp_start 192.168.4.7
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT dhcp_end 192.168.4.73
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT inspection_iprange 192.168.4.128,192.168.4.254
	[stack@director ~]$ crudini --set undercloud.conf DEFAULT scheduler_max_attempts 35
	```

  1. Verify the changes

	```
	[stack@director ~]$ egrep -v "^#|^$" undercloud.conf
	```

  1. Create self-signed certificate for SSL, using the default OpenSSL config file as a template

	```
	[stack@director ~]$ cd /home/stack
	[stack@director ~]$ cp /etc/pki/tls/openssl.cnf /home/stack/templates/undercloud-certs
	[stack@director ~]$ cd /home/stack/templates/undercloud-certs/
	```	

  1. Perform the steps listed in MSC1 if you're using the MSC1 environment in Walnut Creek; alternatively, perform the steps in MSC2 if you're using the MSC2 environment.
	
	Include both IPv6 and IPv4 addresses in the newly created  alt_names section

	### MSC1

	```
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.1 192.168.4.1
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.2 192.168.4.2
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.3 192.168.4.3
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.4 192.168.46.199
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.5  fd00:4888:2000:f801:524:0ff2:0:3
	```
	
	
	### MSC2

	```
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.1 192.168.4.1
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.2 192.168.4.2
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.3 192.168.4.3
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.4 192.168.46.199
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.5 fd00:4888:2000:fc01:524:0ff2:0:3
	```
	
  1. Check parameters

	```
	[stack@director undercloud-certs]$ crudini --get --format ini openssl.cnf ' alt_names '
	```

  1. Set v3_req and v3_ca in openssl.conf

	```
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' v3_req ' subjectAltName @alt_names
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' v3_ca ' subjectAltName @alt_names
	```
  1. Create under cloud key

	```
	[stack@director undercloud-certs]$ openssl genrsa -out undercloud.key.pem 2048
	[stack@director undercloud-certs]$ openssl req -new -config openssl.cnf -key undercloud.key.pem -out undercloud.csr.pem -subj  "/CN=192.168.4.2/C=US/ST=California/L=Walnut Creek/O=Verizon Wireless/OU=HQ Planning"
	```

  1. Create folder for CA cert

	```
	[stack@director undercloud-certs]$ cd /home/stack
	[stack@director ~]$ mkdir -p templates/ca
	[stack@director ~]$ cd templates/ca
	```
	
  1. Create CA cert

	```
	[stack@director ca]$ openssl genrsa -out ca.key.pem 2048
	[stack@director ca]$ openssl req  -key ca.key.pem -new -x509 -days 7300 -extensions v3_ca -out ca.crt.pem -subj "/CN=Walnut Creek Lab CA/C=US/ST=California/L=Walnut Creek/O=Verizon Wireless/OU=HQ Planning"
	[stack@director ca]$ cat << CA > ca.conf
	```

  1. Create ca.conf file

	```
	[stack@director ca]$ cat << EOF > ca.conf
	[ ca ]
	default_ca  	= HQ-PLANNING   # Default CA
	
	[ HQ-PLANNING ]
	private_key 	= ca.key.pem	# CA key
	certificate        = ca.crt.pem	# CA certificate
	database         = ca.db     	# Keep track of certs issued
	serial            	= ca.serial 	# Serial number for next cert
	default_days	= 7300      	# Validity for certs issued
	default_md       = sha256       	# Default message digest
	policy            	= policy_cn 	# What should be in subject
	new_certs_dir   = .         	# Dir for newly created certs
	
	[ policy_cn ]
	commonName  	= supplied  	# CN must be supplied
	EOF
	```

  1. Create CA database and index.txt

	```
	[stack@director ca]$ touch ca.db
	[stack@director ca]$ echo 01 > ca.serial
	[stack@director ca]$ touch index.txt
	[stack@director ca]$ echo "unique_subject = no " > index.txt.attr
	[stack@director ca]$ cd /home/stack/templates/undercloud-certs/
	```
	
  1. Backup and modify openssl.cnf

	```
	[stack@director undercloud-certs]$ cp openssl.cnf openssl.cnf.crudini
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' CA_default ' dir /home/stack/templates/ca
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' CA_default ' certs \$dir
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' CA_default ' crl_dir \$dir
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' CA_default ' database \$dir/index.txt
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' CA_default ' new_certs_dir \$dir
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' CA_default ' certificate \$dir/ca.crt.pem
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' CA_default ' serial \$dir/ca.serial
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' CA_default ' crlnumber \$dir/crlnumber
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' CA_default ' crl \$dir/crl.pem
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' CA_default ' private_key \$dir/ca.key.pem
	[stack@director undercloud-certs]$ crudini --set openssl.cnf ' CA_default ' RANDFILE \$dir/.rand
	```
	
  1. Configure CA certificate 

	```
	[stack@director undercloud-certs]$ openssl ca -config openssl.cnf -extensions v3_req -days 3650 -in undercloud.csr.pem -out undercloud.crt.pem -cert ../ca/ca.crt.pem
	                                                         
	...
	Certificate is to be certified until Jan  3 15:08:15 2027 GMT (3650 days) 
	Sign the certificate? [y/n]:y 
	...                                                                                                                                                        
	1 out of 1 certificate requests certified, commit? [y/n]y 
	Write out database with 1 new entries 
	Data Base Updated 
	```
	
  1. Verify signed cert

	```
	[stack@director undercloud-certs]$ openssl x509 -noout -certopt no_sigdump,no_pubkey -text -in undercloud.crt.pem
	```
	
  1. Combine the undercloud crt and key pem files

	```
	[stack@director undercloud-certs]$ cat undercloud.crt.pem undercloud.key.pem > undercloud.pem
	[stack@director undercloud-certs]$ sudo mkdir /etc/pki/instack-certs
	[stack@director undercloud-certs]$ sudo cp undercloud.pem /etc/pki/instack-certs/
	[stack@director undercloud-certs]$ sudo ln -s /etc/pki/instack-certs/undercloud.pem /etc/pki/ca-trust/source/anchors/undercloud.pem
	[stack@director undercloud-certs]$ sudo update-ca-trust extract
	```
	
  1. Add CA to trust

	```
	[stack@director undercloud-certs]$ cd ../ca
	[stack@director ca]$ sudo cp ca.crt.pem /etc/pki/ca-trust/source/anchors/
	[stack@director ca]$ sudo update-ca-trust extract
	[stack@director ca]$ sudo restorecon -Rv /etc/pki/instack-certs/
	```
	
  1. Make link for undercloud.conf file

	```
	[stack@director ca]$ cd /home/stack
	[stack@director ~]$ ln -s /home/stack/templates/undercloud.conf /home/stack
	```
	
  1. Install undercloud

	```
	[stack@director ~]$ openstack undercloud install
	```
	
  1. Verify installation of undercloud

	```
	[stack@director ~]$ . stackrc
	[stack@director ~]$ nova service list
	```


### Complete Director Configuration
	
Log into the Director server using an ssh client using stack@<director-external-ip> or the iLO remote console as described above. Login using the stack password defined when creating the account.

  1. Download, Extract and Upload the Overcloud images
	
	```
	[stack@director ~]$ source ~/stackrc
	[stack@director ~]$ sudo yum install -y rhosp-director-images rhosp-director-images-ipa
	[stack@director ~]$ cd ~/images
	[stack@director ~/images]$ tar xvf /usr/share/rhosp-director-images/overcloud-full-latest-10.0.tar
	[stack@director ~/images]$ tar xvf /usr/share/rhosp-director-images/ironic-python-agent-latest-10.0.tar
	[stack@director ~/images]$ openstack overcloud image upload --image-path /home/stack/images/
	```
	
	Verify the bm-deploy-ramdisk, bm-deploy-kernel, overcloud-full, overcloud-full-initrd, and overcloud-full-vmliuz images have been uploaded:
	
	```			
	[stack@director ~]$ openstack image list
	```

  2. Verify Ironic and Nova settings are configured to improve behaviour when scaling the deployment

	Check the current status of the existing max_resources_per_stack and scheduler_max_attempts variables in /etc/heat/heat.conf and /etc/nova/nova.conf respectively
			
	```
	[stack@director ~]$ sudo crudini --get /etc/heat/heat.conf DEFAULT max_resources_per_stack
	[stack@director ~]$ sudo crudini --get /etc/nova/nova.conf DEFAULT scheduler_max_attempts
	[stack@director ~]$ sudo crudini --get /etc/nova/nova.conf DEFAULT sync_power_state_interval
	```

	Expect the following results:
	   
	```
	-1
	35
	-1
	```
			
	Tip: If any of these values need to be changed; make sure to restart all OpenStack services before proceeding
	
	```										
	[stack@director ~]$ sudo openstack-service restart
	```
	
## Register Nodes for the Overcloud
	
After OpenStack services have been restarted, log into the Director server using an ssh client using stack@<director-external-ip> or the iLO remote console as described above. Login using the stack password defined when creating the account.

  1. Generate the instackenv.json file

	```
	[stack@director ~]$ cd /home/stack/templates
	[stack@director ~/templates]$ ./genis
	[stack@director ~/templates]$ mv instackenv.json.tmp /home/stack/instackenv.json
	[stack@director ~]$ cd /home/stack/
	```
	
	NOTE: be sure to change the "pm_type" to either "pxe_ipmitool" or "pxe_ilo" depending on your requirements.

  2. Import the nodes into Director

	```
	[stack@director ~]$ source ~/stackrc
	[stack@director ~]$ openstack baremetal instackenv validate
	[stack@director ~]$ openstack baremetal import --json ~/instackenv.json
	```

  3. Configure boot images for nodes

	```
	[stack@director ~]$ openstack baremetal configure boot
	```
	
	Note: This can take over 15 minutes

  3. Verify the correct number of nodes is listed 

	```
	[stack@director ~]$ openstack baremetal node list | egrep -v "^\+|UUID" | wc -l                                                                                      
	```

  4. Set all nodes to manage in preparation for introspection.

	```
	[stack@director ~]$ for node in $(openstack baremetal node list --fields uuid -f value) ; do openstack baremetal node manage $node ; done
	```

  5. Perform introspection on the nodes

	```
	[stack@director ~]$ openstack overcloud node introspect --all-manageable --provide
	```
	
	NOTE:  introspection is expecting only one interface per node to be on the provisioning network.  if more than one nic is found, introspection could attempt to pxe off the wrong interface and fail.
	NOTE:  be sure that the controllers don't have PXE boot enabled for any interface but eno1

	Tip: Check the status of introspection using the following

	```
	[stack@director ~]$ sudo journalctl -l -u openstack-ironic-discoverd -u openstack-ironic-discoverd-dnsmasq -u openstack-ironic-conductor -f
	```
	
	Tip: If any of the nodes fail to introspect; attempt to place them into maintenance mode and perform the introspection manually.

	```
	[stack@director ~]$ ironic node-set-maintenance <uuid-of-bad-node> on
	[stack@director ~]$ openstack baremetal introspection <uuid-of-bad-node> start
	```
	
  6. Set nodes back to available

	```
	[stack@director ~]$ openstack baremetal introspection bulk status | awk '/True/ {print $2}' | xargs -I% openstack baremetal node provide %
	```
	
## Retrieve the Templates from a Git Repository

If your templates reside under version control (for example, git), then simply clone them into place and make the following customizations. If not, included with this document is a vzw-templates-<environment>.tar.xz file, which contains a copy of the templates used generically to build this and similar environments.

Git clone the templates and set up a branch for the environment

	```
	[stack@director ~]$ git clone -b <git-branch> <git-url> ~/templates
	[stack@director ~]$ git branch <new-environment-name>
	[stack@director ~]$ git checkout <new-environment-name>
	```

## Overcloud Certificate Creation

  1. Create directory for overcloud certificate
	
	```
	[stack@director ~]$ cd /home/stack/templates
	[stack@director templates]$ mkdir overcloud-certs
	[stack@director templates]$ cd overcloud-certs
	```

  2. Copy tls-endpoints-public-dns.yaml file to templates folder and openssl.cnf to overcloud-certs folder

	```
	[stack@director overcloud-certs]$ cp templates/my-templates/environments/tls-endpoints-public-dns.yaml ~/templates/  
	[stack@director overcloud-certs]$  cp /etc/pki/tls/openssl.cnf .
```

  3. Modify the newly copied openssl.cnf file

	```
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.1 fd00:4888:2000:fc01:524:ff2:0:6
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.2 192.168.46.202
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' alt_names ' IP.3 192.168.4.6
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' alt_names ' DNS.1 wcmsc2-l-rh-ocld-vip.hqplan.lab
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' CA_default ' dir /home/stack/templates/ca
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' CA_default ' certs \$dir
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' CA_default ' crl_dir \$dir
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' CA_default ' database \$dir/index.txt
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' CA_default ' new_certs_dir \$dir
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' CA_default ' certificate \$dir/ca.crt.pem
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' CA_default ' serial \$dir/ca.serial
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' CA_default ' crlnumber \$dir/crlnumber
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' CA_default ' crl \$dir/crl.pem
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' CA_default ' private_key \$dir/ca.key.pem
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' CA_default ' RANDFILE \$dir/.rand
	[stack@director overcloud-certs]$ crudini --set openssl.cnf ' v3_ca ' subjectAltName @alt_names
	```
	
  4. Create and configure overcloud cert

	```
	[stack@director overcloud-certs]$ openssl genrsa -out overcloud.key.pem 2048
	[stack@director overcloud-certs]$ openssl req -new -config openssl.cnf -key overcloud.key.pem -out overcloud.csr.pem -subj "/C=US/ST=California/L=Walnut Creek/O=Verizon Wireless/OU=HQ Planning/CN=wcmsc2-l-rh-ocld-vip.hqplan.lab"
	[stack@director overcloud-certs]$  openssl ca -config openssl.cnf -extensions v3_req -days 3650 -in overcloud.csr.pem -out overcloud.crt.pem -cert ../ca/ca.crt.pem
	```

  5. Copy tls-endpoints-public-dns.yaml to templates folder

	```
	[stack@director overcloud-certs]$ cd /home/stack/templates/
	[stack@director templates]$ cp my-templates/environments/tls-endpoints-public-dns.yaml .
	```

  6. Modify inject-trust-trust-anchor.yaml file and place in templates folder. 
 
	```
	[stack@director overcloud-certs]$ sed -e '/SSLRootCertificate:/r ca/ca.crt.pem' /home/stack/templates/my-templates/environments/inject-trust-anchor.yaml > /home/stack/templates/inject-trust-anchor.yaml
	[stack@director overcloud-certs]$ sed -i -e '/BEGIN/,/END/s/^/	/g' -e '/The contents of/d' -e 's#\.\./puppet#./my-templates/puppet#' /home/stack/templates/inject-trust-anchor.yaml
	[stack@director overcloud-certs]$ sed -e '/SSLCertificate:/r overcloud-certs/overcloud.crt.pem' -e '/SSLKey:/r overcloud-certs/overcloud.key.pem' /home/stack/templates/my-templates/environments/enable-tls.yaml > /home/stack/templates/enable-tls.yaml
	[stack@director overcloud-certs]$ sed -i -e '/BEGIN/,/END/s/^/	/g' -e '/The contents of/d' -e 's#\.\./puppet#./my-templates/puppet#' /home/stack/templates/enable-tls.yaml
	```

  7. Remove unnecessary information from the enable-tls.yaml file. Example of unneeded information in undercloud.crt.pem. Create sed command.

	```
	[stack@director overcloud-certs]$ 
	```

## Phase 2 - Deploying the Overcloud
### Prepare Director for Overcloud Deployment

  1. Create a script to contain the deploy command. This is a very important step, any minor mistakes here could ruin your deployment and take a very long time to debug.  Do not lose this command, please make sure you have a backup available.

	```
	[stack@director ~]$ cd /home/stack/templates
	[stack@director templates]$ cat << EOF > deploycmd 
	#!/bin/sh -x 
	 
	
	NCOMPUTES=${1}
	TEMPLATES=/home/stack/templates
	STACK_NAME=$(hostname -s|cut -d- -f1)
	
	if [ "x${1}" = "x" ]; then
	    	echo "Usage: ${0} <nb_compute_nodes>" ; exit 127
	fi 
	 
	time openstack overcloud deploy \ 
	--stack ${STACK_NAME} \ 
	--templates ${TEMPLATES}/my-templates \ 
	-e ${TEMPLATES}/environment.yaml \
	-e ${TEMPLATES}/network-environment.yaml \ 
	-e ${TEMPLATES}/storage-environment.yaml \ 
	-e ${TEMPLATES}/enable-tls.yaml \ 
	-e ${TEMPLATES}/inject-trust-anchor.yaml \ 
	-e ${TEMPLATES}/tls-endpoints-public-dns.yaml \ 
	-e ${TEMPLATES}/extraconfig-environment.yaml \ 
	-e ${TEMPLATES}/environment-rhel-registration.yaml \ 
	-e ${TEMPLATES}/rhel-registration-resource-registry.yaml \ 
	--control-scale 3 \  
	--compute-scale ${NCOMPUTES} \ 
	--control-flavor baremetal \ 
	--compute-flavor baremetal \ 
	--ntp-server fd00:4888:2000:1::26 
	EOF
	```

  2. Verify capabilities of the nodes

	```
	[stack@director ~]$ source stackrc
	[stack@director ~]$ openstack baremetal node list --long -f json | jq '.[] | .Name + " " + .Properties.capabilities'
	```
	
  3. Copy the payload script ::temporary step::

	```
	[stack@director ~]$ cp <source folder>/rebuild-node-payload.sh ~/templates/rebuild-node-payload.sh
	```

  4. Verify values for external ips in 30-IPv4 file

	```
	[stack@director ~]$ grep IPV.= ~/templates/node-payload/scripts/controller/30-IPv4
	```
	
	Should contain values corresponding to the environment:

	```
	IPV4="192.168.46.202"
	IPV6="fd00:4888:2000:fc01:524:ff2:0:6"
	```

	If not set them using sed

	```
	[stack@director ~]$ sed -i 's/IPV4=".*"/IPV4="<director-external-ipv4-ip>"/' ~/templates/node-payload/scripts/controller/30-IPV4
	[stack@director ~]$ sed -i 's/IPV6=".*"/IPV4="<director-external-ipv6-ip>"/' ~/templates/node-payload/scripts/controller/30-IPV4
	```

  5. Run rebuild-node-payload.sh ::temporary step::

	```
	[stack@director ~]$ ~/templates/rebuild-node-payload.sh
	```

  6. Make sure the cloud name is correct in the environment.yaml file. 

	```
	...
	CloudName: 'wcmsc2-l-rh-ocld-vip.hqplan.lab'
	...
	```
	
	Note: Make sure the files for tls and inject-trust.yaml are ascii

### Run the Deploy Command

  1. Verify the capabilities have been set correctly:	
	```
	[stack@director ~]$ openstack baremetal list --long -f json | jq '.[] | .Name + " " + .Properties.capabilities'
	```

  2. Make the script executable and execute the deploy command specifying 3 compute nodes, 3 controller nodes are specified in the script. We'll eventually scale up to 32 compute nodes once this deployment is successful. 

	Note: If the overcloud is being replaced make sure all remnants on the storage array are cleared out before the command is run.

	```
	[stack@director ~]$ chmod +x deploycmd
	[stack@director ~]$ ./deploycmd 3
	```

	It will take a while for the deploy to finish ( ~ 1 Hour), but while that's happening, feel free to check your progress using the suggestions below.
	
	Tip: If you would like to monitor the progress of your deployment, use one of the below options. It can often be advantageous to couple these with the watch command so you don't have execute them multiple times.
	
	Check the status of the deployment steps.
	
	```
	[stack@director ~]$ source ~/stackrc
	[stack@director ~]$ openstack stack list --nested | grep -v COMP
	[stack@director ~]$ openstack stack event list <Stack Name>
	```
						
	Check the status of your baremetal provisioning 
	
	```
	[stack@director ~]$ ironic node-list
	[stack@director ~]$ ironic node-show <node-uuid>
	```
	
	Check the status of Ironic steps (most notably in power management)
	
	```
	[stack@director ~]$ sudo journalctl -u openstack-ironic-conductor -u openstack-ironic-api
	```

	Check Heat and Nova configurations
	
	```
	[stack@director ~]$ heat resource-list -n9 overcloud
	[stack@director ~]$ heat resource-show overcloud <failed-resource>
	[stack@director ~]$ nova list
	[stack@director ~]$ nova show <node-uuid>
	```

	As root user, you can check the status of the Nova, Heat, and Ironic logs 
	
	```
	[stack@director ~]$ su - root
	[root@director ~]$ less /var/log/nova/<nova-{scheduler,compute,conductor}.log>
	[root@director ~]$ less /var/log/heat/<heat-{engine,api}.log>
	[root@director ~]$ less /var/log/ironic/<ironic-{api,conductor}.log>
	```

  5. Now that you've verified the deployment has been successful on the 3 controller and 3 compute nodes, expand the cluster to include all 32 compute nodes; much like the earlier step, this will take some time ( >30 minutes ) 

	```
	[stack@director ~]$ ./deploycmd 32
	```
  6. Install storops library on the controllers. Repeat for each controller. 
 
	```
	[stack@director ~]$ ssh -f -N -L8080:10.194.169.108:9090 root@10.255.252.22
	[root@ocld0 ~]$ sudo -i
	[root@ocld0 ~]# export http_proxy=localhost:8080
	[root@ocld0 ~]# export https_proxy=localhost:8080
	[root@ocld0 ~]# curl -s -O https://bootstrap.pypa.io/get-pip.py
	[root@ocld0 ~]# python get-pip.py
	[root@ocld0 ~]$ exit
	```

  7. Once you've verified that the stack has been successfully deployed, verify that the default route exists. 

	```
	[stack@director ~]$ source stackrc
	[stack@director ~]$ nova list | awk '/wcmsc/{print substr($12,10.15)}' | xargs -I% ssh % "echo -n \$(hostname -s); echo -n ': '; /sbin/ip -6 r | grep def" 2>/dev/null
	```

  8. Disable strict host key checking for the stack user to be able to access overcloud nodes unprompted. Change the ownership and mode of the new config file you've created

	```
	[stack@director ~]$ echo -e "Host 192.168.4.* wc* ocld* cmp*\n\tUser heat-admin\n\tStrictHostKeyChecking no" >> /home/stack/.ssh/config
	[stack@director ~]$ chmod 600 /home/stack/.ssh/config
	```
	
  9. Include the hostnames of the overcloud nodes in /etc/hosts 

	```
	[stack@director ~]$ nova list | awk '/ACT/ {split($12,x,"="); split($4,y,"-"); printf("%-15s %s.hqplan.lab %s %s%s\n", x[2], $4, $4, y[4], y[5])}' | sort -t- -k 4,4r -k5,5n | sudo tee -a /etc/hosts
	```
	
  10. Set all nodes in the cluster to standby mode so we can perform further checks and configurations after deployment. 

	```
	[stack@director ~]$ echo -n ocld{0..2} | xargs -d' ' -I% ssh % "sudo systemctl stop openstack-* neutron-*"
	[stack@director ~]$ ssh ocld0 sudo pcs cluster standby --all
	```
	
  11. Monitor the status of the pcs cluster and wait until all services are stopped and the entire cluster is in standby mode.

	* NOTE: The watch command will only terminate when Ctrl-C is sent to the terminal. It will continue to output pcs status even after the entire cluster has been put in standby mode. You should only have to wait about 5 minutes for this to complete.

	```
	[stack@director ~]$ ssh ocld0
	[heat-admin@ocld0 ~]$ watch -n1 'sudo pcs status | grep stop'
	[heat-admin@ocld0 ~]$ exit
	```

  12. Update the controller nodes. 

	```
	[stack@director ~]$ echo -n ocld{0..2} | xargs -d' ' -P3 -I% ssh % "sudo yum update -y"
	```
	
  13. Reboot the controller nodes to apply the changes.  

	```
	[stack@director ~]$ source stackrc
	[stack@director ~]$ nova list | awk '/ocld/{print $4}' | xargs -I% nova reboot %
	```

  14. Take the controllers out of standby mode 

	```
	[stack@director ~]$ ssh ocld0 ?sudo pcs cluster unstandby --all?
	```
	
  15. Check the status of the controllers to verify they have been taken out of standby mode.

	```
	[stack@director ~]$ source wcmsc2rc
	[stack@wcmsc2-l-rh-ucld-01 ~]$ cinder service-list
	[stack@wcmsc2-l-rh-ucld-01 ~]$ glance image-list
	[stack@director ~]$ echo -n ocld{0..2} | xargs -d' ' -P3 -I% ssh % "systemctl --failed"
	```
	
  16. Kill all the openstack-nova-compute services on the compute nodes and update them all.

	```
	[stack@director ~]$ source ~/stackrc
	[stack@director ~]$ nova list | awk '/cmp/{print $4}' | xargs -I% ssh % "sudo yum update -y" 
	```

  17. Reboot the compute nodes to apply changes

	```
	[stack@director ~]$ nova list | awk '/cmp/{print $4}' | xargs -I% nova reboot %
	```
	
  18. Verify that the default IPv6 route exists after reboot.  

	```
	[stack@director ~]$ nova list | awk '/wcmsc/{print substr($12,10.15)}' | xargs -I% ssh % "echo -n \$(hostname -s); echo -n ': '; /sbin/ip -6 r | grep def" 2>/dev/null
	```

  19. Verify the all the nodes and networks are in a good state. 

	```
	[stack@director ~]$ source ~/wcmsc2rc
	[stack@director ~]$ nova service-list
	[stack@director ~]$ neutron agent-list
	```
	
## Phase 3 - Configuring the Overcloud
### Create Provider Networks

  1. Create the Provider Subnets	
	* NOTE: if this site will contain vSMSC VNFs, it's necessary that the IPv4 subnet be first in neutron net-list. They are sorted by uuid, so it may take multiple tries to create the subnet with a uuid lower than the other networks

				
	```
	[stack@director ~]$ source ~/<cloud-name>rc
	[stack@director ~]$ neutron net-create <network-name>  --provider:network_type vlan --router:external true --provider:physical_network datacentre --provider:segmentation_id <network-vlan-id> --shared
	```
	
  2. Create the Provider Subnets	
	* NOTE: if this site will contain vSMSC VNFs, it's necessary that the IPv4 subnet be first in neutron net-list. They are sorted by uuid, so it may take multiple tries to create the subnet with a uuid lower than the other networks
				
	```
	[stack@director ~]$ neutron subnet-create <network-name> <network-address/prefix> --name <subnet-name> --disable-dhcp --gateway 
	```

### Configure SR-IOV Networks
	
  1. Create the SR-IOV Networks

	```
	[stack@director ~]$ source ~/<cloud-name>rc
	[stack@director ~]$ neutron net-create sriov_a --provider:physical_network=sriov_a --provider:network_type flat --shared
	[stack@director ~]$ neutron net-create sriov_b --provider:physical_network=sriov_b --provider:network_type flat --shared
	```

  2. Create the SR-IOV Subnets

	```
	[stack@director ~]$ neutron subnet-create sriov_a --name subnet-sriov_a fd00:a::/32 --disable-dhcp --ip-version 6
	[stack@director ~]$ neutron subnet-create sriov_b --name subnet-sriov_b fd00:b::/32 --disable-dhcp --ip-version 6
	```

### Configure SR-IOV NIC Agent

Log into the Director server using an ssh client using stack@<director-external-ip> or the iLO remote console as described above:

  1. Add SR-IOV configuration to overcloud controllers

	```
	[stack@director ~]$ echo -n ocld{0..2} | xargs -d' ' -I% ssh % "sudo openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_sriov supported_pci_vendor_devs 8086:10ed"
	[stack@director ~]$ echo -n ocld{0..2} | xargs -d' ' -I% ssh % "sudo openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_sriov agent_required True"
	```
	
  2. Restart neutron-server resource

	```
	[stack@director ~]$ ssh ocld0 "sudo pcs resource restart neutron-server-clone"
	```
	
  3. Add SR-IOV NIC Agent configuration to computes

	```
	[stack@director ~]$ echo -n cmp{0..31} | xargs -d' ' -I% ssh % "sudo openstack-config --set /etc/neutron/plugins/ml2/sriov_agent.ini sriov_nic physical_device_mappings sriov_a:ens1f0,sriov_a:ens2f0,sriov_b:ens1f1,sriov_b:ens2f1"
	```
	
  4. Install the SR-IOV NIC Agent package

	```
	[stack@director ~]$ sudo yum install --downloadonly --downloaddir=/var/www/html openstack-neutron-sriov-nic-agent
	[stack@director ~]$ sudo ln -s /var/www/html/openstack-neutron-sriov-nic-agent-{8.1.2-4.el7ost,latest}.noarch.rpm
	[stack@director ~]$ echo -n cmp{0..31} | xargs -d' ' -I% -P16 ssh % "sudo yum install -y -q http://192.168.4.1/openstack-neutron-sriov-nic-agent-latest.noarch.rpm
	```
	
  5. Enable and start SR-IOV NIC Agent service
	
	```
	[stack@director ~]$ echo -n cmp{0..31} | xargs -d' ' -I% -P16 ssh % "sudo systemctl enable neutron-sriov-nic-agent"
	[stack@director ~]$ echo -n cmp{0..31} | xargs -d' ' -I% -P16 ssh % "sudo systemctl start neutron-sriov-nic-agent"
	```

### Configure Fencing

Log into the Director server using an ssh client using stack@<director-external-ip>.

  1. Source ~/stackrc, make it executable, and run the fencing.sh script.

	```
	[stack@director ~]$ source ~/stackrc
	[stack@director ~]$ chmod +x fencing.sh
	[stack@director ~]$ ./fencing.sh
	```

# Appendix I - Troubleshooting Techniques

## Reversion - Rollback packages and reboot if necessary

Occasionally, package updates can cause problems when running an older version of OSP Director (often with various C libraries, etc.). When such problems arise, it's often necessary to perform a yum history rollback, which has the exact opposite functionality of yum update. Often times this can be a useful method to remedy issues caused by a failed overcloud deploy.


  1. Determine the ID of each package that was updated. The date and line should match the steps performed to which you would like to roll back.

	```
	[heat-admin@controller1 ~]$ sudo yum history list all
	```

  1. Then rollback the update 

	```
	[heat-admin@controller1 ~]$ sudo yum history rollback <ID>
	```

	If reverting a kernel or libraries affecting many processes (like glibc), reboot the server:        
  
	```
	[heat-admin@controller1 ~]$ sudo systemctl reboot 
	```
  
## Troubleshooting a failed overcloud deploy
In most cases, a failed overcloud deploy with exit with CREATE_FAILED or UPDATE_FAILED.

You will want to identify precisely what step it failed with by looking for the failed resources:

  1. Identify the failed steps 
  
	```
	[stack@director ~]$ heat stack-list --show-nested | grep FAIL
	```
  
  1. After identifying a failed sub-stack, look for more detail in its resource list

	```
	[stack@director ~]$ heat resource-list <sub-stack-name>
	```
  
  1. In this sub-stack, one or more resources will typically have failed. Look for further details in their output, as in:

	<failed-resource-name> is typically one of 'NovaCompute' or some integer value.
	
	```
	[stack@director ~]$ heat resource-show <sub-stack-name> <failed-resource-name>
	```
	
	For resources of type SoftwareDeployment (not the similarly named SoftwareDeployments), there is an additional method of obtaining log information:
	
	```
	[stack@director ~]$ heat deployment-output-show <resource-uuid> deploy_stdout
	```
  
## Deleting a failed stack
  
  1. Attempt regular deletion of the stack

	```
	[stack@director ~]$ heat stack-delete <stack-name>
	```
  
  1. Check for orphaned nested stacks

	It's sometimes possible for stacks to not properly delete their nested stack resources properly. When this occurs, even though the stack is gone, there may be lingering resources. This typically happens when a stack resource is deleted outside heat. One example of this is deleting a nova instance that heat had created, without using heat stack-delete. To show all stacks, you pass the --show-nested flag to heat stack-list
  
	```
	[stack@director ~]$ heat stack-list --show-nested
	```
	
  1. Deleting orphaned stacks

	```
	[stack@director ~]$ heat stack-delete <orphaned-stack-name>
	```

  1. Cleaning Ironic
  
	If not all nodes are off, some deployments may fail due to not being able to successfully control the power state of the nodes. In most cases, manual intervention is not necessary. Before performing this step, ensure that all nodes are accessible by their IPMI interfaces from the director.
	
	```
	[stack@director ~]$ ironic node-list | awk '/power on/ {print $2}' | xargs -I% ironic node-set-power-state % off
	```
	
	After ensuring that all nodes are properly set to the power off state, you can then proceed with the deploy.
	
## Interrupting stack creation

If you should, for any reason need to interrupt the stack creation (such as noticing that a value is clearly incorrect and will cause the stack to fail), you can interrupt stack operations in the following manner

#### WARNING: This is potentially destructive to the state of the stack, only execute this if you are sure you will be deleting the stack as a next step

This can be performed by restarting the heat engine service

```        
[stack@director ~]$ sudo systemctl restart openstack-heat-engine
```	

## Stack Failure Due to Post Deployment Validation

If the stack fails when validating the nodes. There is a ping condition in /home/stack/templates/my-templates/validation-scripts/all-nodes.sh that could fail if the first ping times out. A workaround we put in place is to add a ping command before the condition statement. This should catch the dropped packet if any. 

```
…
  # added to get rid of first dropped ping
  ping -c 2 $GW
  
  if ! ping -c 1 $GW &> /dev/null; then
    echo "FAILURE"
    echo "$GW is not pingable."
    exit 1
  fi
…
```

# Appendix II - Post-deployment Configurations

## Create accounts and tenants

### Create OpenStack Tenants
#### * NOTE: "Projects" and "Tenants" are one in the same here. In newer versions of OSP, the "Tenant" moniker will be dropped        

  1. Run the following for every tenant you would like to create. 
  
	```
	[stack@director ~]$ source ~/overcloudrc
	[stack@director ~]$ openstack project create <tenant-name> --description "<tenant-description>”
	```

### Create OpenStack Users
   
  1. Run the following to create a user
  
	```
	[stack@director ~]$ source ~/overcloudrc
	[stack@director ~]$ openstack user create --password=<user-password> --email=<email-address> <username>
	[stack@director ~]$ openstack role add --user <username> --project <tenant-id> <user-role>
	```
  
  1. Now add a role to that user

	```
	[stack@director ~]$ openstack role add --user <username> --project <tenant-id> <user-role>
	```
  
  Repeat steps 1 and 2 for each additional user you would like to create 


## Injecting files into the overcloud images
#### * Note: these modifications only take effect for newly deployed nodes, it will not retroactively update existing nodes. Additionally, it may be changed by the overcloud deploy or update scripts after injection.
  
  1. Create the file you want to inject 

	```
	[stack@director ~]$ cat << EOF > <file-name> 
	<file-content>
	EOF
	```
  
  1. Use virt-copy-in to inject the file

	This results in the file being placed in the directory specified at the end of the previous command. For example, virt-copy-in -a image.qcow2 foo /tmp/ would result in the image containing a file /tmp/foo with the contents of the file foo from the local directory.

	```
	[stack@director ~]$ virt-copy-in -a <image> foo <path-inside-image>
	```

  1. Use virt-customize to run commands inside the image

	For customizations like grub, it's necessary to run things like grub2-mkconfig, this can be done with virt-customize

	```
	[stack@director ~]$ virt-customize -a <image> --run-command '<command-to-run>'
	```
	
## Creating a host aggregate/availability zone

  1. Create the aggregate (and optionally an availability zone)

	```
	[stack@director ~]$ nova aggregate-create <aggregate-name> [availability-zone-name]
	```
  
  1. Add hosts to the aggregate

	```
	[stack@director ~]$ nova aggregate-add-host <host>
	```
  
  1. Set metadata on the aggregate

	```
	[stack@director ~]$ nova aggregate-set-metadata <key>=<value> [...]
	```
	#### * NOTE: You are able to set more than one metadata attribute at once.

## Create Aggregates and Modify Pinning Configuration

There should be a script under /home/stack/templates/util called create_aggregates.sh

  1. Modify the PATTERN variable to correspond to the current installation
	#### * NOTE: The command below is just an example, please modify this according to your installation.
	
	```
	[stack@director ~]$ vi ~/templates/util/create_aggregates.sh
	
	...
	PATTERN=$(hostname -s)-c-pr-cmp
	...
	```

  1. Modify the compute ranges if necessary so that the proper number of compute nodes are included in each aggregate
  
	```
	[stack@director ~]$ vi ~/templates/util/create_aggregates.sh
	...
	for i in ${PATTERN}-{<compute-start-index>,<compute-end-index>}.localdomain; do
	  nova aggregate-add-host <aggregate-name>-<{A,B}> $i &
	  nova aggregate-add-host <aggregate-name> $i &
	done
	...
	```
  
  1.  Run the create_aggregates.sh script

	```
	[stack@director ~]$ ~/templates/util/create_aggregates.sh
	```

  1. Create the aggregates

	```
	[stack@director ~]$ nova aggregate-create <aggregate-name> <az-name>
	```                        
  
	Below is a table containing the current list of zones: 
	
	| Aggregate | Availability Zone |
	| --- | --- |
	| Active-A | Zone-A |
	| Active-B | Zone-B |
	| Active-Pin-Isolate-A |	Zone-PI-A |
	| Active-Pin-Isolate-B | Zone-PI-B |
	| Maintenance-A | Zone-M-A |
	| Maintenance-B | Zone-M-B |

  1. Update the pinning configuration 

	```
	[stack@director ~]$ ssh <compute-node> “sudo openstack-config --set /etc/nova/nova.conf DEFAULT vcpu_pin_set <pinned-cpus>”
	[stack@director ~]$ ssh <compute-node> “sudo systemctl restart openstack-nova-compute”
	```
  
## Update Ironic Nodes to Boot from Disk

Since the nodes no longer need to boot from PXE (for a reprovision, ironic would set them back to PXE automatically), and on HP hardware, if booting from a NIC where PXE is not present, there can be a situation where the node will fail to boot successfully, it's recommended to change to booting from disk instead.

  1. Source the stack user's credentials 

	```
	[stack@director ~]$ source ~/stackrc
	```
  
  1. Change the boot device for each node 

	```
	[stack@director ~]$ ironic node-list | awk '/power/{print $2}' | xargs -I% ironic node-set-bootdevice % disk
	```
  
## Modifying policy.json to Include the network_admin Role
                
  1. Return to the director node and execute the following:

	```
	[stack@director ~]$ openstack role create network_admin
	```
  
  1. To add the role to a user, execute the following:

	```
	[stack@director ~]$ openstack role add --user <user> --project <project> network_admin
	``` 

## Registering the Overcloud Nodes with subscription-manager

  1. Create a script to register all the overcloud nodes
  
	```
	[stack@director ~]$ cat << EOF > overcloud_reg.sh 
	#!/bin/sh
	  
	source /home/stack/stackrc
	
	for i in $(nova list | awk '/ACTIVE/{print $4}')
	do
	  echo "Registering $i"
	  echo "fd00:4888:2000:1::26                   cawcrhsd1v.hqplan.lab cawcrhsd1v" | ssh $i "sudo tee -a /etc/hosts"
	  ssh $i "sudo yum install -y http://cawcrhsd1v/pub/katello-ca-consumer-latest.noarch.rpm"
	  ssh $i "sudo subscription-manager register --org='Default_Organization' --activationkey='nec1'"
	  ssh $i "sudo yum install -y katello-agent"
	done
	EOF
	```
  
  1. Execute the script
	
	```
	[stack@director ~]$ ./overcloud_reg.sh
	```

## Configuring the Ironic Rules Model
  
  1. Obtain the ironic-inspector token and set an environment variable for it.

	```
	[stack@director ~]$ IPW=$(sudo openstack-config --get /etc/ironic-inspector/inspector.conf keystone_authtoken admin_password)
	```
  
  1. Obtain the UUID for your node by running the following
	
	```
	[stack@director ~]$ swift -U service:ironic -K $IPW list ironic-inspector
	```
  
	Expect output resembling the following
	
	```
	extra_hardware-e8bd1a03-6e08-477d-aa0d-c58b2900e1e7
	extra_hardware-eada6616-d4df-4d2f-b99f-7e832ad8b98d
	extra_hardware-f88a7465-9c47-4d3e-b652-c6da0fedadc5
	extra_hardware-fd5504c1-713b-4d5d-920e-f1ba3fa47618
	inspector_data-0d71257d-b142-440d-a7b3-4ebe863e1a03
	inspector_data-0f00c0bd-2c5a-498d-9af8-0370bbe9022f
	inspector_data-0fa34e2f-a508-4f5b-b7c3-15c5676ae433
	...
	```
  
  1. Download the hardware details for each node type. Do this for both compute and controller UUIDs.

	```
	[stack@director ~]$ swift -U service:ironic -K $IPW download ironic-inspector inspector_data-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
	```
  
  1. Verify the correct node details have been downloaded

	```
	[stack@director ~]$ jq '.' inspector_data-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX | less
	[stack@director ~]$ jq '.inventory.system_vendor.product_name' inspector_data-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
	```
  
  1. Create a rules model that includes node and chassis details

	```JSON
	[stack@director ~]$ cat << EOF > rules-model.json
	[
	    {
	      "description": "<controller-node-description>",
	      "conditions": [
	        {
	           "op": "eq",
	           "field": "data://inventory.system_vendor.product_name",
	           "value": "<make-and-model-of-controller-hardware>"
	         }
	      ],
	      "actions": [
	        {
	          "action": "set-capability",
	          "name": "profile",
	            "value": "control"
	         }
	      ]
	    },
	    {
	      "description": "<compute-node-description>",
	      "conditions": [
	        {
	          "op": "eq",
	          "field": "data://inventory.system_vendor.product_name",
	            "value": "<make-and-model-of-compute-hardware>"
	        }
	      ],
	      "actions": [
	        {
	          "action": "set-capability",
	          "name": "profile",
	          "value": "compute"
	        }
	    ]
	    }
	]
	EOF
	```
  
  1. Import the rules model

	```
	[stack@director ~]$ openstack baremetal introspection rule import ~/rules-model.json
	```
  
## Update the Controller Packages
        
Log into the Director server using an ssh client using stack@<director-external-ip> or the iLO remote console as described above. Source stackrc and perform the following steps:
  
  1. Stop all openstack-nova-compute service on all compute nodes

	```
	[stack@director ~]$ nova service list --binary nova-compute | awk '/enabled/{print $6}' | xargs -I% -P32 ssh % sudo systemctl stop openstack-nova-compute
	```
  
  1. Standby the controllers to prevent cluster failures

	```
	[stack@director ~]$ ssh ocld0 "sudo pcs cluster standby --all"; sleep 25
	```
  
  1. Verify the controllers are down

	```
	[stack@director ~]$ ssh ocld0 "sudo pcs status | grep Started"
	```
  
  1. Verify the controller nodes are running the same kernel version

	```
	[stack@director ~]$ for host in $(nova list | awk '/cmp/{print substr($12,10,15)}'); do echo ${host}; ssh ${host} "sudo grep linux16 /boot/grub2/grub.cfg | head -1 | cut -d\  -f2"; done
	```
  
  1. Clean the metadata and update packages on the controllers

	```
	[stack@director ~]$ for node in $(nova list | grep -v Name | grep -v cmp | awk '{print $4}'); do ssh -i ~/.ssh/id_rsa heat-admin@$node sudo yum clean metadata; done
	[stack@director ~]$ for node in $(nova list | grep -v Name | grep -v cmp | awk '{print $4}'); do ssh -i ~/.ssh/id_rsa heat-admin@$node sudo yum update -y; done
	```
  
  1. If installing a new kernel or libraries affecting many processes (like glibc), reboot the servers:

	```
	[stack@director ~]$ nova list | awk '/ocld/{print $4}' | xargs -I% nova reboot % 
	```

	#### Note: before proceeding make sure you can ssh to all controller nodes. If the code below returns server name and time then server is up.
  
	```
	[stack@director ~]$ for node in $(nova list | grep -v Name | grep -v cmp | awk '{print $4}'); do ssh -i ~/.ssh/id_rsa heat-admin@$node hostname; ssh -i ~/.ssh/id_rsa heat-admin@$node date; done
	```
  
  1. Re-join the controllers to the HA cluster

	```
	[stack@director ~]$ for node in $(nova list | grep -v Name | grep -v cmp | awk '{print $4}'); do ssh heat-admin@$node sudo pcs cluster unstandby $node; done
	```
  
  1. Check cluster status (the first fifteen lines or so should provide enough information)

	```
	[stack@director ~]$ for node in $(nova list | grep -v Name | grep -v cmp | awk '{print $4}'); do ssh heat-admin@$node sudo pcs status | head -15; done
	```

	Verify that all services show as started on each controller. Repeat for each controller.
	
	```
	[stack@director ~]$ ssh ocld0 'sudo pcs status | grep pace'
	[stack@director ~]$ ssh ocld0 'sudo pcs status | grep stonith'
	[stack@director ~]$ ssh ocld0 'sudo pcs property | grep stonith'
	```
	
	Tip: In the event the services are not started; perform a resource cleanup and check the status of the services again.
	
	```
	[stack@director ~]$ ssh heat-admin@<broken-controller>
	[heat-admin@<broken-controller> ~]$ sudo pcs resource cleanup
	[heat-admin@<broken-controller> ~]$ sudo pcs status
	```	
	
## Update the Compute Packages

Log into Director server using an ssh client stack@<director-external-ip> or the iLO remote console as described above. Login using the stack user, source stackrc and the perform the following steps:

  1. Verify that all the compute nodes are running the same kernel version
	
	```
	[stack@director ~]$ for host in $(nova list | awk '/cmp/{print substr($12,10,15)}'); do echo ${host}; ssh ${host} "sudo grep linux16 /boot/grub2/grub.cfg | head -1 | cut -d\  -f2"; done
	```
  
  1. Update packages on each compute node. Code will update 15 instances at the same time. Remove or modify -P15 if you want more or less to update at the same time. 

	```
	[stack@director ~]$ nova list | awk '/cmp/{print $4}' | xargs -I% -P15 ssh heat-admin@% sudo yum update -y
	```
  
  1. Reboot the nodes in case crucial libraries (like glibc) or a new kernel were installed during update: 

	```
	[stack@director ~]$ nova list | awk '/cmp/{print $4}' | xargs -I% nova reboot %
	```
	
  1. Verify the compute nodes have checked back in 
	
	```
	[stack@director ~]$ source ~/overcloudrc
	[stack@director ~]$ watch -n15 “nova service-list”
	```

	The compute node should show in the table with nova-compute binary and an “up” state. It might take some time for the nodes to come back to “up” state. If a node stays in “down” state, troubleshoot and/or open a support ticket or follow reversion steps.
	
                         

