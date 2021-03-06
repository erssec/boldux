---
layout: post
title:  "Exploring Microsoft Exchange for penetration testing and red teaming - part 1"
date:   2020-08-03 19:55:40 +0200
categories: exchange
---
Microsoft Exchange is among the most used e-mail servers. The Active Directory integration and flexibility are among the selling points of this widespread product of Microsoft. In this blog post, I will explore Exchange Mailbox Server and cover various attack surfaces, methodologies, and tools for Red Teaming operations and penetration tests.

Most, if not every, security professionals have seen the familiar Exchange login page, often found on the mail.domain.tld subdomain. The login page shows the exchange server’s version used. The familiar yellow color of Exchange 2003 for example should be a trigger for most engagements. But for this blog post, I will dive into the recent versions of Exchange and dive a bit deeper than searching for known RCEs or vulnerabilities and using those.

The Exchange front-end (the webserver) has various functionalities and endpoints. Although a lot of information could be gathered from light enumeration of Exchange about the internal network, abusing Exchange elevates to a whole other level when credentials are available. Let’s say you are in a red teaming operation. Password spraying is an easy way to gather credentials. Even when 2FA is enabled, some attacks mentioned in this series may still work. Password spraying on Exchange has become easy with tools like ruler or atomizer.py. Besides this, leaked credentials often are a great source of information. Users may reuse leaked passwords. This is another way of finding valid credentials, also using the bruteforce tools mentioned above.

When looking at the Microsoft Exchange directory on a Mail server, at 'C:\Program Files\Microsoft\Exchange Server\V15', we can get a better look at the different endpoints. Here, we see various directories with the needed executables and configuration files. Since this blog focuses on the accessible endpoints from an external perspective, the 'ClientAccess' and 'frontend\httpproxy' directories are the most interesting one for now because they reviel our attack surface.



Looking at the endpoints, we can identify the following major parts of Exchange:
* Autodiscover
* ecp
* mapi
* OAB
* Owa
* Powershell
* powershell-proxy
* rest API
* rpc
* sync

However, I could not find a particular endpoint in these directories, which is Microsoft ActiveSync. In this part 1, I will focus on Microsoft ActiveSync with a bit of Autodiscover. The following parts will cover each endpoint with the attack surface, possible tools that could assist you in your operations, and the code behind certain interacties. 

###Autodiscover
Let’s start with Autodiscover, which most clients use to get configuration settings from the exchange server. Autodiscover is meant for client applications so that minimal user interaction is required to configure things like address books and certain endpoints. This works in single forest and multi-forest environments. Autodiscover uses POST requests for requesting configurations. We can send a request to '/Autodiscover/Autodiscover.xml' or '.svc', depending on the access method used (POX or SOAP respectively). In my experience, .xml most often works.
For authentication, basic authentication is possible. A base64 encoded header with 'domain\user:password' should be provided if you are crafting the request yourself. In the body of the POST request, the following XML should be provided (ptsecurity.com). The request should look like this.


Under some circumstances, a browser user agent might not work. Hence, I recommend using an application user agent. Hereabove, I used the Microsoft Word user agent. 
The response reveals a lot of information. In the X-BackEndCookie we can see the SID of the user. More interestingly, we can see the Domain Controller under the AD tags. This is particularly interesting for the next section, where we can explore SMB shares using Exchange. The internal Exchange server’s hostname is mentioned as well, under the PublicFolderServer tag. The OABurl tag reveals the OAB, which includes the address book. Since this is Exchange, this reveals all the Exchange users, most often also a huge part of the Active Directory. This is particularly interesting for further password spraying. For more information on OAB and the attack paths, please take a loot at ptsecurity’s blog. 




###Microsoft ActiveSync
ActiveSync, as the name suggests, is the Exchange synchronization protocol based on HTTP and XML. Mobile phones could access information on the Exchange Server by ActiveSync. 
Since the Exchange credentials are Active Directory credentials, we can abuse ActiveSync with a tool called PEAS by F-SecureLabs. This python2 tool enables us to run commands on the ActiveSync enabled Exchange server. PEAS enables us to enumerate SMB shares of hosts inside the network (also the ones without exposed SMB shares to the internet!), but only hosts without dots (.) in their name, strangely.

From the Autodiscover response, we saw that the Domain Controller is DC-1. Every user should be able to see the 'NETLOGON' and 'SYSVOL' shares. So, let’s try to enumerate those. This might not be a good idea for OpSec reasons, so please consider the benefit before using this in a real engagement.
With the option '-u' and '-p', we can specify the credentials. '--list-unc' we can specify the share or host we want to enumerate. This is how it looks like:
'python2.7 -m peas -u 'domain\username' -p 'password' exchangeserver.domain.com --list-unc '\\dc-1\''

!image(/assets/img/blog/activesync-dc1.png)

Maybe enumerating the DC is not the first thing you would do. It might be more beneficial to enumerate workstations or printers for files, password.txts, or other sensitive information! Active Directory domains often have a specific file server for the user profiles and home directories. Once a privileged user's password is compromised, no phishing or injection is required to access sensitive information. An attacker could simply browse all SMB shares without even touching a C2 server.

!image(/assets/img/blog/activesync-workstation.png)

With the --dl-unc option, we can download the files enumerated. This way, we have access to the internal network without no foothold in it. 

This wraps up the first part of Exploring Exchange series. In the next parts, the powershell endpoint and OAB will be covered!

ruler: https://github.com/sensepost/ruler
atomizer.py: https://github.com/byt3bl33d3r/SprayingToolkit/blob/master/atomizer.py
ptsecurity’s blog: https://swarm.ptsecurity.com/attacking-ms-exchange-web-interfaces/
peas: https://github.com/FSecureLABS/PEAS
F-SecureLabs: https://labs.f-secure.com/archive/accessing-internal-fileshares-through-exchange-activesync/
