Veritas Cluster Server Cheatsheet

Raw
veritas_cluster_server.md
Veritas Cluster Server Cheatsheet
LLT & GAB
VCS uses two components, LLT and GAB, to share data over the private networks among systems. These components provide the performance and reliability required by VCS.
Name	Description
LLT	LLT (Low Latency Transport) provides fast, kernel-to-kernel comms and monitors network connections. The system admin configures the LLT by creating a configuration file (llttab) that describes the systems in the cluster and private network links among them. The LLT runs in layer 2 of the network stack.
GAB	GAB (Group membership and Atomic Broadcast) provides the global message order required to maintain a synchronised state among the systems, and monitors disk comms such as that required by the VCS heartbeat utility. The system admin configures GAB driver by creating a configuration file ( gabtab).
LLT and GAB files

Name	Description
/etc/llthosts	The file is a database, containing one entry per system, that links the LLT system ID with the hosts name. The file is identical on each server in the cluster.
/etc/llttab	The file contains information that is derived during installation and is used by the utility lltconfig.
/etc/gabtab	The file contains the information needed to configure the GAB driver. This file is used by the gabconfig utility.
/etc/VRTSvcs/conf/config/main.cf	The VCS configuration file. The file contains the information that defines the cluster and its systems.

gabconfig	-c Configure the driver for use
	-n Number of systems in the cluster.
	-a Ports currently in use
-x Manually seed node (careful when manually seeding as you can create 2 separate clusters)

LLT and GAB Commands
Name	Description
Verifying that links are active for LLT	lltstat -n
verbose output of the lltstat command	lltstat -nvv | more
open ports for LLT	lltstat -p
display the values of LLT configuration directives	lltstat -c
lists information about each configured LLT link	lltstat -l
List all MAC addresses in the cluster	lltconfig -a list
stop the LLT running	lltconfig -U
start the LLT	lltconfig -c
verify that GAB is operating	gabconfig -a
	Note: port a indicates that GAB is communicating, port h indicates that VCS is started.
stop GAB running	gabconfig -U
start GAB	gabconfig -c
start GAB with a set number of nodes	gabconfig -c -n <number of nodes>
override the seed values in the gabtab file	gabconfig -c -x
GAB Port Memberbership
Name	Description
List Membership	gabconfig -a
Unregister port f	/opt/VRTS/bin/fsclustadm cfsdeinit
Port Function	a gab driver
	b I/O fencing (designed to guarantee data integrity)
	d ODM (Oracle Disk Manager)
	f CFS (Cluster File System)
	h VCS (VERITAS Cluster Server: high availability daemon)
	o VCSMM driver (kernel module needed for Oracle and VCS interface)
	q QuickLog daemon
	v CVM (Cluster Volume Manager)
	w vxconfigd (module for cvm)
	y kernel-tokernel communication used for I/O shipping.

Cluster daemons
Name	Description
High Availability Daemon	had
Companion Daemon	hashadow
Resource Agent daemon	<resource>Agent
Web Console cluster management daemon	CmdServer
Cluster Log Files
Name	Description
Log Directory	/var/VRTSvcs/log
primary log file (engine log file)	/var/VRTSvcs/log/engine_A.log
Agent logs	/var/VRTSvcs/log/_A.log

Starting and Stopping the cluster
Name	Description
"-stale" instructs the engine to treat the local config as stale	hastart [-stale|-force]
"-force" instructs the engine to treat a stale config as a valid one
Bring the cluster into running mode from a stale state using the configuration file from a particular server.	hasys -force <server_name>
Stop the cluster on the local server.	hastop -local
Note: This will also bring any clustered resources offline.
Stop cluster on local server but evacuate (failover) the application/s to another node within the cluster.	hastop -local -evacuate
Stop the cluster on all nodes but leave the clustered resources online.	hastop -all -force

Cluster Status
Name	Description
display cluster summary	hastatus -summary
continually monitor cluster	hastatus
verify the cluster is operating	hasys -display
Cluster Details
Name	Description
information about a cluster	haclus -display
value for a specific cluster attribute	haclus -value <attribute>
modify a cluster attribute	haclus -modify <attribute name>
Enable LinkMonitoring	haclus -enable LinkMonitoring
Disable LinkMonitoring	haclus -disable LinkMonitoring

Users
Name	Description
add a user	hauser -add <username>
modify a user	hauser -update <username>
delete a user	hauser -delete <username>
display all users	hauser -display

System Operations
Name	Description
add a system to the cluster	hasys -add <sys>
delete a system from the cluster	hasys -delete <sys>
Modify a system attributes	hasys -modify <sys> <modify options>
list a system state	hasys -state
Force a system to start	hasys -force
Display the systems attributes	hasys -display [-sys]
List all the systems in the cluster	hasys -list
Change the load attribute of a system	hasys -load <system> <value>
Display the value of a systems nodeid (/etc/llthosts)	hasys -nodeid
Freeze a system (No offlining system, No groups onlining)	hasys -freeze [-persistent][-evacuate]
	Note: main.cf must be in write mode
Unfreeze a system ( reenable groups and resource back online)	hasys -unfreeze [-persistent]
	Note: main.cf must be in write mode

Dynamic Configuration
The VCS configuration must be in read/write mode in order to make changes. When in read/write mode the configuration becomes stale, a .stale file is created in $VCS_CONF/conf/config. When the configuration is put back into read-only mode the .stale file is removed.
Name	Description
Change configuration to read/write mode	haconf -makerw
Change configuration to read-only mode	haconf -dump -makero
Check what mode cluster is running in	haclus -display | grep -i 'readonly'
	0 = write mode
	1 = read only mode
Check the configuration file	hacf -verify /etc/VRTS/conf/config
	Note: you can point to any directory as long as it has main.cf and types.cf
convert a main.cf file into cluster commands	hacf -cftocmd /etc/VRTS/conf/config -dest /tmp
convert a command file into a main.cf file	hacf -cmdtocf /tmp -dest /etc/VRTS/conf/config


Service Groups
Name	Description
add a service group	haconf -makerw
	  hagrp -add groupw
	  hagrp -modify groupw SystemList sun1 1 sun2 2
	  hagrp -autoenable groupw -sys sun1
	haconf -dump -makero
delete a service group	haconf -makerw
	  hagrp -delete groupw
	haconf -dump -makero
change a service group	haconf -makerw
	  hagrp -modify groupw SystemList sun1 1 sun2 2 sun3 3
	haconf -dump -makero
	
	Note: use the "hagrp -display " to list attributes
list the service groups	hagrp -list
list the groups dependencies	hagrp -dep <group>
list the parameters of a group	hagrp -display <group>
display a service group's resource	hagrp -resources <group>
display the current state of the service group	hagrp -state <group>
clear a faulted non-persistent resource in a specific grp	hagrp -clear <group> [-sys] <host> <sys>
Change the system list in a cluster.	# remove the host
	hagrp -modify grp_zlnrssd SystemList -delete <hostname>
	
	# add the new host (don't forget to state its position)
	hagrp -modify grp_zlnrssd SystemList -add <hostname> 1
	
	# update the autostart list
	hagrp -modify grp_zlnrssd AutoStartList <host> <host>
Service Group Operations
Name	Description
Start a service group and bring its resources online	hagrp -online <group> -sys <sys>
Stop a service group and takes its resources offline	hagrp -offline <group> -sys <sys>
Switch a service group from system to another	hagrp -switch <group> to <sys>
Enable all the resources in a group	hagrp -enableresources <group>
Disable all the resources in a group	hagrp -disableresources <group>
Freeze a service group (disable onlining and offlining)	hagrp -freeze <group> [-persistent]
	note: use the following to check "hagrp -display <group> | grep TFrozen"
Unfreeze a service group (enable onlining and offlining)	hagrp -unfreeze <group> [-persistent]
	note: use the following to check "hagrp -display <group> | grep TFrozen"
Enable a service group. Only Enabled groups can be brought online.	haconf -makerw
	  hagrp -enable <group> [-sys]
	haconf -dump -makero
	
	Note to check run the following command "hagrp -display | grep Enabled"
Disable a service group. Stop from bringing online.	haconf -makerw
	  hagrp -disable <group> [-sys]
	haconf -dump -makero
	Note to check run the following command "hagrp -display | grep Enabled"
Flush a service group and enable corrective action.	hagrp -flush <group> -sys <system>


Resources
Name	Description
add a resource	haconf -makerw
	  hares -add appDG DiskGroup groupw
	  hares -modify appDG Enabled 1
	  hares -modify appDG DiskGroup appdg
	  hares -modify appDG StartVolumes 0
	haconf -dump -makero
delete a resource	haconf -makerw
	  hares -delete <resource>
	haconf -dump -makero
change a resource	haconf -makerw
	  hares -modify appDG Enabled 1
	haconf -dump -makero
	
	Note: list parameters "hares -display <resource>"
change a resource attribute to be globally wide	hares -global <resource> <attribute> <value>
change a resource attribute to be locally wide	hares -local <resource> <attribute> <value>
list the parameters of a resource	hares -display <resource>
list the resources	hares -list
list the resource dependencies	hares -dep

Resource Operations
Name	Description
Online a resource	hares -online <resource> [-sys]
Offline a resource	hares -offline <resource> [-sys]
display the state of a resource( offline, online, etc)	hares -state
display the parameters of a resource	hares -display <resource>
Offline a resource and propagate the command to its children	hares -offprop <resource> -sys
Cause a resource agent to immediately monitor the resource	hares -probe <resource> -sys <sys>
Clearing a resource (automatically initiates the onlining)	hares -clear <resource> [-sys]


Resource Types
Name	Description
Add a resource type	hatype -add <type>
Remove a resource type	hatype -delete <type>
List all resource types	hatype -list
Display a resource type	hatype -display <type>
List a partitcular resource type	hatype -resources <type>
Change a particular resource types attributes	hatype -value <type> <attr>

Resource Agents
Name	Description
add a agent	pkgadd -d . <agent package>
remove a agent	pkgrm <agent package>
change a agent	n/a
list all ha agents	haagent -list
Display agents run-time information i.e has it started, is it running ?	haagent -display <agent_name>
Display agents faults	haagent -display

Resource Agent Operations
Name	Description
Start an agent	haagent -start <agent_name>[-sys]
Stop an agent	haagent -stop <agent_name>[-sys]

From <https://gist.github.com/sorquan/186d9d645b1e82417dc5a1c976ace416> 
http://www.unixstage.com/2018/06/13/vcs-cheat-sheet-complete/
