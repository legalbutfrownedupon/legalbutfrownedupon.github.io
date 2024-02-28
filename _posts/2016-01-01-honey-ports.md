---
layout: post
title: "HoneyPorts"
date: 2016-01-01 00:00:00 -0000
image:
  path: /assets/images/hp_banner.png
  alt: Honey Ports
---

## History
HoneyPorts was originally released as an opensource project by Paul Asadoorian of PaulDotCom.  The original code can be found here: [HoneyPorts-0.4](https://code.google.com/archive/p/honeyports/source)

## New Features in 0.5
- Multihreaded
- Runs on multiple ports
- Set firewall rules to expire after a given time
- Config file
- Custom port messages
- Whitelist
- Auto-whitelist of local IPs
- Output logging
- Interactive command mode
- List and Flush Firewall rules while running

## Usage
HoneyPorts is now multithreaded.  This means you can run HoneyPorts on multiple ports at once.  Once HoneyPorts is running, you can issues commands at any time.  This interactive command mode allows users to complete tasks without having to exit the HoneyPorts program.
### Runtime Commands:
-p   Port numbers. Single port number or comma separated.
-v   Verbose mode. Shows more information at runtime such as version number, whitelist, output log file location and config file location.
-o   Specify a location for the output log file.  without this option, HoneyPorts will take the output log file location from the config file.
-c   specify a location for the config file.  Without this option, HoneyPorts will look for the config file in the current working directory.
-e   Firewall rules expire.  Specify a time in minutes for the firewall rules to be removed.
-h   Display usage.
### Interactive Mode Commands:
q or quit      Exit the program
h or help     List all Interactive Mode Commands
fwlist            List all firewall rules created
fwflush        Flush all firewall rules created

These commands can be issued at any time once HoneyPorts has launched.
  
Examples:
Basic usage of HoneyPorts running on ports 21 and 23.
![Desktop View](/assets/images/hp_1.png) 

Attempting to run HoneyPorts on a port that is currently being used by another service (SSH).  HoneyPorts will continue to run on successful ports.
![Desktop View](/assets/images/hp_2.png) 

Running in verbose mode and also using -e to flush all firewall rules every 60 minutes.
![Desktop View](/assets/images/hp_3.png) 

Setting custom locations for both the config file and the output log file.  Using these commands at run-time take precedence over values in the config file.
![Desktop View](/assets/images/hp_4.png) 

Connection attempt was made from a whitelisted host.
![Desktop View](/assets/images/hp_5.png) 

Running the h command once the program has started will list all possible commands that can be used in command mode.
![Desktop View](/assets/images/hp_6.png) 

fwlist command shows there are no firewall rules.  A connection attempt is made from 10.0.0.5.  A firewall rule is created.  Running fwlist again shows the new firewall rule.  Running the fwflush command clears all firewall rules.  Running fwlist yet again verifies that the fiewall rule has been flushed.
![Desktop View](/assets/images/hp_7.png) 

Similar example as above but this time the firewall rules were cleared via the timer set with the -e command at run-time.  No user interaction was needed to clear the firewall rules in this case.
![Desktop View](/assets/images/hp_8.png) 

Running the -h option will display the usage which lists all run-time commands as well as all interactive mode commands.
![Desktop View](/assets/images/hp_9.png) 

## Config File
HoneyPorts needs a config file to operate.  By default, HoneyPorts will look in the current working directory for hpconfig.conf.  Also, users can use the -c option during run-time to specify a different location of the config file.  The config file is used to store default locations for the output log file, set custom banners for each port to send attackers and add IP addresses to the whitelist.

Default Config File:
```
###########################################
#
#  HoneyPorts Config File  
#
###########################################

##############Output Logs##################
#Location for the log file to be stored.
#Leave blank (default) to log data to the current working directory of the application.
#Use outputLoglin for Linux, use outputLogwin for Windows and use outputLogmac for Mac.
#
#Examples:
#outputLoglin: \home\adminitrator\hplog.txt
#outputLogwin: C:\Users\admin\hplog.txt
#outputLogmac: \Users\admin\hplog.txt

[logs]
outputLoglin:
outputLogwin:
outputLogmac:


############Custom Banners#################
#Use port 0 for a default message. This message will be sent if there is not a custom message for the port used in a connection.
#Use the format '[port number]':'[message]' and comma separate the entries.

[banners]
banners = {
	'0':'=====Default Message=====',
	'21':'---message for port 21---',
	'23':'---message for port 23---'
	}


############Whitelist######################
#comma separate entries into the whitelist without whitespace
#Example:
#whitelist: 10.0.0.10,192.168.1.100,192.168.1.101

[whitelist]
whitelist: 
```

## Output Log Location
The first section lists the output log location for each operating system.  Leaving this field blank for the OS in use will cause the output log to be stored in the current working directory of the application.  You can also use the -o option while running HoneyPorts to specify an output log location.  Using the -c option from the command line will overwrite the settings from the config file.

## Custom Banners
The Custom Banners section can be used to serve back custom messages depending on what port a connection is made on.  Use port 0 as the default message.  If HoneyPorts is listening on a port which does not have a defined message in this section, the message for port 0 will be served to the attacker.

## Whitelist
Adding IPs to this whitelist will cause the program to not create firewall rules when a connection is made from that IP.  These IPs should be comma separated without spaces.  There is no need to add local IPs to this list.  That will happen automatically when the program is loaded.

## Output Log
The Output Log file will store information about all connection attempts while HoneyPorts is running.

Example Log Entries:
```
06/17/14 17:33:37 Connection attempt from 10.0.0.5 on port: 22
Host was whitelisted. No firewall rule created.

06/17/14 17:34:38 Connection attempt from 10.0.0.6 on port: 23
A Windows Firewall rule was created.
```

## Project Code
Code for HoneyPorts can be found [here](https://github.com/legalbutfrownedupon/honeyports)
