---
layout: post
title: "Customizing Powershell Empire to Avoid Detection"
date: 2016-06-17 00:00:00 -0000
image:
  path: /assets/images/empire_banner.png
  alt: Empire C2 Opsec
---

Powershell Empire is a very powerful post-exploitation framework for Windows environments. The tool has been gaining popularity since its release in 2015. As more red teams and malicious threat actors utilize the tool, more detection is being developed to identify the use of Empire on the network. This post will show some customizations that change the network traffic of Empire in order to avoid detection.

Before we get into what changes can be made to alter Empire traffic, it is important to understand the Empire staging process and what data is in the network traffic at each stage.

For this example, we will be using the Launcher stager which is a Powershell one-liner that will perform an IEX on a file hosted on Empire’s web server in order to pull down the stage 2 payload. More detailed information about the staging process can be found on the Empire website at: [Empire](https://github.com/EmpireProject/Empire)

Most common network intrusion devices will look to identify common components in the staging process. Those items can include the user agent string, the name of the files being accessed, the protocol and the port number. There will be different artifacts at different stages of the staging process that should be changed. We will first take a look at custom profiles and how to use them while creating a listener.

There are several demo profiles included in the data/profiles directory. The format of an Empire profile is much simpler than a profile for Cobalt Strike. The profiles for Empire allow you to change the payload file name, the user agent string as well as headers in the communication. Creating a custom profile can be used to emulate a particular threat actor to test your defenses or customize the tool to avoid detection. Our goal is the latter so we will be creating a profile that will attempt to look like normal user activity. Depending on the attack infrastructure you have setup and the domain name that you are using, you could potentially make your traffic look like traffic analytics, browsing activity or connectivity to a streaming service.

By showing information while setting up a listener, you can see the default profile that is used by Empire.  
![Desktop View](/assets/images/emp-1.png)  
The default profile has a set of default file names which include:
- /admin/get.php
- /news.asp
- /login/process.jsp

There are the file names that the Empire agent will communicate with via GET and POST requests to check for tasks and upload data.

We will simply replace these files with new file names in our custom profile.

We will also change the UAS in our custom profile. If you have already performed your recon, you may have a better idea of what browsers are deployed in the target environment. The goal here is to avoid detection so setting our C2 to use the same UAS as most traffic in the environment is another way to blend in to the thousands of requests going across the wire.

For this example we are going to set the UAS as “Mozilla/5.0 (profileUAS)”. This is so when we look at the traffic we will be able to see where our custom UAS is being utilized.

This is the custom profile we have so far:
“/wp_includes/test1.php|Mozilla/5.0 (profileUAS)|Accept:*/*”

This will use the file /wp_includes/test1.php and a UAS of Mozilla/5.0 (profileUAS)

When creating our Empire listener, we will set that in the DefaultProfile option. Rather than changing the values of the default profile while creating the listener, you can make the changes in setup_database.py before creating the empire database. This will replace the default value so all listeners you create will have your custom profile by default.

Lets fire off our payload and get a shell to take a look at the traffic.

We execute our payload on the victim machine and the first request back to our attacking box is the following:
![Desktop View](/assets/images/emp-2.png)  
As you can see, this does not have our custom file name or our custom UAS.

The second request is the following POST request:  
![Desktop View](/assets/images/emp-3.png)  
Again, we are seeing default values for the file names and UAS rather than our custom data.

The next request is the stage 2 request which again uses default data.

Finally once the actual C2 traffic starts, we see our custom data in the network traffic:  
![Desktop View](/assets/images/emp-4.png)  
So during the rest of the engagement, our traffic will be using the custom values we provided in the profile. This does not help us if we are detected during the download of Stage 1, the download of the encryption keys or the download of Stage 2. We will need to do more work to customize those three requests in order to avoid detection during the staging process.

When we created our launcher stager after we started our listener, there was an option for Useragent String. We will set that as Mozilla/5.0 (StagerUAS) so we can see in the traffic where it is being used.  
![Desktop View](/assets/images/emp-5.png)  
If you base64 decode the payload you will be able to see that the UAS has been updated but we still see the index.asp reference in the URL.

Looking at the traffic now, we see that the UAS has been updated to our custom UAS for the three remaining requests (Stage 1 download, crypto keys download, and Stage 2 download).  
![Desktop View](/assets/images/emp-6.png)  
But we still have that index.asp which will be very easy for signatures to detect. These are the default files we are still seeing in our attack traffic:
- GET /index.asp – Stage 0
- POST /index.jsp – Stage 1
- POST /index.php – Stage 2

To change these files, we will have to dig into Empire’s database located at /opt/Empire/data/empire.db. The easiest way we can make changes to this database is to make changes to the setup/setup_database.py file and then rebuild the database by running the python file.  
![Desktop View](/assets/images/emp-7.png)  
If you want to edit the database without rebuilding it, you can open the database with sqlitebrowser. Once open, you can see in the config table, there are entries for stage0_uri, stage1_uri and stage2_uri. Which represent the sage that is using that URI.  
![Desktop View](/assets/images/emp-8.png)  
We can input custom filenames into the database to take care of the staging URIs then write our changes to the database. Note, you will have to restart Empire after making changes to the database for the changes to take effect.

Since these are actual hosted files which contain data for the staging process, they cannot be the same name, that is why the creators of Empire decided to use the same file name but change the file extension. We will want to change that as well since it would be trivial for a NIDs device to look for traffic to a new domain or IP and make requests to the same file name with three different extensions in a row. Most web applications wont use asp, jsp and php all in the same app with the same filename.

Again, depending on your pretext, you may want to change them to a random file like analytics5f2c8d534a0 to make it look like analytics traffic or name them about.asp, about_us.asp and home.asp if you are making it look like web browsing.

For this case I will just name them one.asp, two.asp and three.asp so we can see in the pcaps that we are seeing our custom data.  
![Desktop View](/assets/images/emp-9.png)  
As you can see, the staging process has been updated with our custom filenames.

Another default setting that can could easily be detected is the default port number of 8080. Again, this goes back to having a solid pretext but it is important to change the port number of your listener to match the pre-text. 80 or 443 might blend in better with all the other traffic taking place on the network.

TL;DR
There are multiple values in multiple locations that will need to be changed to truly customize your Empire C2 traffic to avoid common detections. You will want to change the following items:

- C2 file path and name in the profile used in your listener
- UAS in the profile used in your listener
- Port number used in your listener
- UAS used in your Launcher Stager
- File names used by stage 0, stage 1 and stage 2 in the database by editing the setup_database.py file or editing the empire.db file

There will be more customizations that can made to further make your traffic blend in and avoid detection. These simple changes however should prove useful to avoiding most common signature based detections looking for Empire traffic.
