---
layout: post
title: "Getting the Most Out of Cobalt Strike’s Browser Pivot"
date: 2019-12-18 09:00:00 -0500
categories: research
image:
  path: /assets/images/browser-pivot-header/png
  alt: Cobalt Strike Browswer Pivot
---

The socks proxy and browser pivot features of Cobalt Strike are great tools to be able to access internal resources during a red team operation. While the browser pivot is a man-in-the-browser attack, the socks proxy can be used for protocols other than http/https such as RDP. This post will focus on the web access capabilities and when each should be used.

A common scenario is that operators have breached the perimeter and have an internal foothold in the network. We now want to be able to access internal web resources such as intranets and applications. Depending on the user we have compromised and other creds we may have at our disposal, we can choose to use either a browser pivot or a socks proxy.

The socks proxy allows operators to connect into the target network with other tools such as a web browser. Since this connection does not take advantage of WinINet, it will not inherit the credentials of the compromised user. This allows operators to use credentials of other users through an unrelated foothold in the network. An example of this is using a compromise on user A’s machine to login to internal applications as user B. If you want to use another user’s credentials through the compromise, a socks proxy is the option you want to use.

The browser pivot is a feature works by injecting the functionality into an Internet Explorer tab so it can access the WinINet authentication and session data. This means that if a user is running Internet Explorer, you can inject into a tab and access the cookies that user has to access the same site they are logged in to. So if you do not have a different user’s credentials, the browser pivot is the option you want to use.

One thing I commonly hear Red Team Operators say is that they do not use the browser pivot feature due to victims rarely using IE and if they are using IE, the site they are using is of little value to the operation. This makes sense as most examples and documentation I have seen on the feature show a compromised user with IE open and accessing the target application or site at the time of browser pivot use. I agree that this is a rare scenario to encounter. I have found a technique to utilize the amazing functionality of the CS browser pivot even if the user does not use IE or not currently accessing a sensitive web applications.

The first issue we need to overcome is that the compromised user is not using IE at the time of compromise. If a tab of IE is not running, the browser pivot is not usable (Browser Pivot only works in IE).

![Desktop View](/assets/images/browser-pivot-no-ie.png)  
_Browswer Pivot with no IE process running_

We can use powerpick or psinject to create a new com object of the IE application. This will spawn the iexplore.exe process on the target machine. This process also will spawn a child iexplor.exe process that the browser pivot can be injected into automatically. All of this is not visible to the user on the compromised machine. They would have to open task manager to see that iexplore.exe is running. The command to create the IE process in the background is as follows:

```
powerpick $ie = new-object -com "InternetExplorer.Application"
```

After running the command, we have the IE broker process and the injectable tab processes running.

![Desktop View](/assets/images/browser-pivot-ie-proc.png)  

![Desktop View](/assets/images/browser-pivot-ie-proc2.png)  
_IE processes running in CS’s process listing and and windows task manager_

After running this command, we can create a browser pivot and inject it into the newly created IE tab. If the organization is using SSO, there is a good chance that this browser pivot will allow access to all internal applications that SSO would authenticate due to the WinINet access.

![Desktop View](/assets/images/browser-pivot-with-ie.png)  
_Browser Pivot is now available in our new process_

A common attack workflow would be:

- Use a recon tool such as SeatBelt or perform manual searches to identify the user’s browser history, bookmarks, favorites to build a list of internal targets.
- Use the powershell command above to create the IE process in the background
- Inject the browser pivot into the newly created IE process
- Setup your browser with an http proxy to connect to the browser pivot
- Brows to target URLs and IPs and see which assets you have access to

Hopefully this post will help you as an operator utilize the browser pivot functionality during on-net operations even when IE is not being used during a compromise.

As always, read the CS documentation on these features to fully understand their functionality.

<https://cobaltstrike.com/help-browser-pivoting>
