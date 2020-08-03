---
layout: post
title:  "Exploring Microsoft Exchange for penetration testing and red teaming - part 1"
date:   2020-08-03 19:55:40 +0200
categories: exchange
---
Microsoft Exchange is among the most used e-mail servers. The Active Directory integration and flexibility are among the selling points of this widespread product of Microsoft. In this blog post, I will explore Exchange Mailbox Server and cover various attack surfaces, methodologies, and tools for Red Teaming operations and penetration tests.

Most, if not every, security professionals have seen the familiar Exchange login page, often found on the mail.domain.tld subdomain. The login page shows the exchange server’s version used. The familiar yellow color of Exchange 2003 for example should be a trigger for most engagements. But for this blog post, I will dive into the recent versions of Exchange and dive a bit deeper than searching for known RCEs or vulnerabilities and using those.

The Exchange front-end (the webserver) has various functionalities and endpoints. Although a lot of information could be gathered from light enumeration of Exchange about the internal network, abusing Exchange elevates to a whole other level when credentials are available. Let’s say you are in a red teaming operation. Password spraying is an easy way to gather credentials. Even when 2FA is enabled, some attacks mentioned in this series may still work. Password spraying on Exchange has become easy with tools like [ruler] or [atomizer.py]. Besides this, leaked credentials often are a great source of information. Users may reuse leaked passwords. This is another way of finding valid credentials, also using the bruteforce tools mentioned above.

When looking at the Microsoft Exchange directory on a Mail server, at 'C:\Program Files\Microsoft\Exchange Server\V15', we can get a better look at the different endpoints. Here, we see various directories with the needed executables and configuration files. Since this blog focuses on the accessible endpoints from an external perspective, the 'ClientAccess' and 'frontend\httpproxy' directories are the most interesting one for now because they reviel our attack surface.

{% highlight ruby %)
 Directory of C:\Program Files\Microsoft\Exchange Server\V15\frontend\httpproxy

09/03/2020  15:21    <DIR>          .
09/03/2020  15:21    <DIR>          ..
09/03/2020  15:20    <DIR>          autodiscover
09/03/2020  15:10    <DIR>          bin
09/03/2020  15:20    <DIR>          ecp
09/03/2020  15:21    <DIR>          ews
09/03/2020  15:20    <DIR>          mapi
09/03/2020  15:21    <DIR>          oab
09/03/2020  15:20    <DIR>          owa
09/03/2020  15:21    <DIR>          powershell
09/03/2020  15:21    <DIR>          pushnotifications
09/03/2020  15:20    <DIR>          ReportingWebService
09/03/2020  15:20    <DIR>          rest
09/03/2020  15:20    <DIR>          rpc
09/03/2020  15:21         1,790,951 SharedWebConfig.config
09/03/2020  15:21    <DIR>          sync
{% endhighlight %}

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

Request
{% highlight ruby %}
POST /autodiscover/autodiscover.xml HTTP/1.1
Host: yourexchangeserver
User-Agent: Microsoft Office/16.0 (Windows NT 10.0; Microsoft Outlook 16.0.10730; Pro)
Authorization: Basic base64encodedcredentials
Content-Length: 341
Content-Type: text/xml

<Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/requestschema/2006">
    <Request>
      <EMailAddress>myemail@domain.com</EMailAddress>
      <AcceptableResponseSchema>http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a</AcceptableResponseSchema>
    </Request>
</Autodiscover>
{% endhighlight %}

Response

{% highlight ruby %}
HTTP/1.1 200 OK
Cache-Control: private
Content-Type: text/xml; charset=utf-8
Server: Microsoft-IIS/10.0
request-id: 611b13b0-48ff-473f-a4fc-723a8e9540b1
X-CalculatedBETarget: mx-1.internal.local //here, we can find the internal mail server's hostname
X-DiagInfo: MX-1 
X-BEServer: MX-1
X-AspNet-Version: 4.0.30319
Set-Cookie: X-BackEndCookie=S-1-5-21-3865823697-1816233505-1834004910-1139=u56Lnp2ejJqBz8nOy5mcmcbSysfJytLLnZ3I0seezsvSysrLys3GzsvOzMydgYHNz83P0s/G0s/Nq8/Gxc/PxcvOgZyGnZqNnZCLlpzRlpCBzw==; expires=Wed, 02-Sep-2020 09:00:41 GMT; path=/autodiscover; secure; HttpOnly //here, we can see the SID of the authenticated domain user
X-Powered-By: ASP.NET
X-FEServer: MX-1
Date: Mon, 03 Aug 2020 09:00:41 GMT
Content-Length: 3834

<?xml version="1.0" encoding="utf-8"?>
<Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/responseschema/2006">
  <Response xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a">
    <User>
      <DisplayName>firstname lastname</DisplayName>
      <LegacyDN>/o=domain.com/ou=Exchange Administrative Group (FYDIBOHF23SPDLT)/cn=Recipients/cn=00c6f832386f4dc8a75f157293078ea0-firstname lastname</LegacyDN>
      <AutoDiscoverSMTPAddress>user@cdomain.com</AutoDiscoverSMTPAddress>
      <DeploymentId>502b6157-6152-4ec6-8d53-17a57596a09c</DeploymentId>
    </User>
    <Account>
      <AccountType>email</AccountType>
      <Action>settings</Action>
      <MicrosoftOnline>False</MicrosoftOnline>
      <Protocol>
        <Type>EXCH</Type>
        <Server>2f07aa42-d2b8-4d45-86b0-4a5208a2e771@domain.com</Server>
        <ServerDN>/o=domain.com/ou=Exchange Administrative Group (FYDIBOHF23SPDLT)/cn=Configuration/cn=Servers/cn=2f07aa42-d2b8-4d45-86b0-4a5208a2e771@domain.com.io</ServerDN>
        <ServerVersion>73C18779</ServerVersion>
        <MdbDN>/o=domain.com/ou=Exchange Administrative Group (FYDIBOHF23SPDLT)/cn=Configuration/cn=Servers/cn=2f07aa42-d2b8-4d45-86b0-4a5208a2e771@domain.com.io/cn=Microsoft Private MDB</MdbDN>
        <PublicFolderServer>mx-1.domain.com.io</PublicFolderServer>
        <AD>dc-1.domain.com.io</AD>	//under the AD tag, we can find the domain controller's hostname
        <ASUrl>https://mx-1.domain.com.io/EWS/Exchange.asmx</ASUrl>
        <EwsUrl>https://mx-1.domain.com.io/EWS/Exchange.asmx</EwsUrl>
        <EmwsUrl>https://mx-1.domain.com.io/EWS/Exchange.asmx</EmwsUrl>
        <EcpUrl>https://mx-1.domain.com.io/owa/</EcpUrl>
        <EcpUrl-um>?path=/options/callanswering</EcpUrl-um>
        <EcpUrl-aggr>?path=/options/connectedaccounts</EcpUrl-aggr>
        <EcpUrl-mt>options/ecp/PersonalSettings/DeliveryReport.aspx?rfr=olk&amp;exsvurl=1&amp;IsOWA=&lt;IsOWA&gt;&amp;MsgID=&lt;MsgID&gt;&amp;Mbx=&lt;Mbx&gt;&amp;realm=domain.com.io</EcpUrl-mt>
        <EcpUrl-ret>?path=/options/retentionpolicies</EcpUrl-ret>
        <EcpUrl-sms>?path=/options/textmessaging</EcpUrl-sms>
        <EcpUrl-photo>?path=/options/myaccount/action/photo</EcpUrl-photo>
        <EcpUrl-tm>options/ecp/?rfr=olk&amp;ftr=TeamMailbox&amp;exsvurl=1&amp;realm=domain.com.io</EcpUrl-tm>
        <EcpUrl-tmCreating>options/ecp/?rfr=olk&amp;ftr=TeamMailboxCreating&amp;SPUrl=&lt;SPUrl&gt;&amp;Title=&lt;Title&gt;&amp;SPTMAppUrl=&lt;SPTMAppUrl&gt;&amp;exsvurl=1&amp;realm=domain.com.io</EcpUrl-tmCreating>
        <EcpUrl-tmEditing>options/ecp/?rfr=olk&amp;ftr=TeamMailboxEditing&amp;Id=&lt;Id&gt;&amp;exsvurl=1&amp;realm=domain.com.io</EcpUrl-tmEditing>
        <EcpUrl-extinstall>?path=/options/manageapps</EcpUrl-extinstall>
        <OOFUrl>https://mx-1.domain.com.io/EWS/Exchange.asmx</OOFUrl>
        <UMUrl>https://mx-1.domain.com.io/EWS/UM2007Legacy.asmx</UMUrl>
        <OABUrl>https://mx-1.domain.com.io/OAB/eca3066f-9c48-4ad7-85e1-eeaf13d5352f/</OABUrl>
        <ServerExclusiveConnect>off</ServerExclusiveConnect>
      </Protocol>
      <Protocol>
        <Type>EXPR</Type>
        <Server>mx-1.domain.com.io</Server>
        <SSL>Off</SSL>
        <AuthPackage>Ntlm</AuthPackage>
        <ServerExclusiveConnect>on</ServerExclusiveConnect>
        <CertPrincipalName>None</CertPrincipalName>
        <GroupingInformation>Default-First-Site-Name</GroupingInformation>
      </Protoco>
      <Protocol>
        <Type>WEB</Type>
        <Internal>
          <OWAUrl AuthenticationMethod="Basic, Fba">https://mx-1.domain.com.io/owa/</OWAUrl>
          <Protocol>
            <Type>EXCH</Type>
            <ASUrl>https://mx-1.domain.com.io/EWS/Exchange.asmx</ASUrl>
          </Protocol>
        </Internal>
      </Protocol>
    </Account>
  </Response>
</Autodiscover>
{% endhighlight %}

Under some circumstances, a browser user agent might not work. Hence, I recommend using an application user agent. Hereabove, I used the Microsoft Word user agent. 
The response reveals a lot of information. In the X-BackEndCookie we can see the SID of the user. More interestingly, we can see the Domain Controller under the AD tags. This is particularly interesting for the next section, where we can explore SMB shares using Exchange. The internal Exchange server’s hostname is mentioned as well, under the PublicFolderServer tag. The OABurl tag reveals the OAB, which includes the address book. Since this is Exchange, this reveals all the Exchange users, most often also a huge part of the Active Directory. This is particularly interesting for further password spraying. For more information on OAB and the attack paths, please take a loot at [ptsecurity’s blog]. 




###Microsoft ActiveSync
ActiveSync, as the name suggests, is the Exchange synchronization protocol based on HTTP and XML. Mobile phones could access information on the Exchange Server by ActiveSync. 
Since the Exchange credentials are Active Directory credentials, we can abuse ActiveSync with a tool called [PEAS] by [F-SecureLabs]. This python2 tool enables us to run commands on the ActiveSync enabled Exchange server. PEAS enables us to enumerate SMB shares of hosts inside the network (also the ones without exposed SMB shares to the internet!), but only hosts without dots (.) in their name, strangely.

From the Autodiscover response, we saw that the Domain Controller is DC-1. Every user should be able to see the 'NETLOGON' and 'SYSVOL' shares. So, let’s try to enumerate those. This might not be a good idea for OpSec reasons, so please consider the benefit before using this in a real engagement.
With the option '-u' and '-p', we can specify the credentials. '--list-unc' we can specify the share or host we want to enumerate. This is how it looks like:
'python2.7 -m peas -u 'domain\username' -p 'password' exchangeserver.domain.com --list-unc '\\dc-1\''

![image](/assets/img/blog/activesync-dc1.png)

Maybe enumerating the DC is not the first thing you would do. It might be more beneficial to enumerate workstations or printers for files, password.txts, or other sensitive information! Active Directory domains often have a specific file server for the user profiles and home directories. Once a privileged user's password is compromised, no phishing or injection is required to access sensitive information. An attacker could simply browse all SMB shares without even touching a C2 server.

![image](/assets/img/blog/activesync-workstation.png)

With the --dl-unc option, we can download the files enumerated. This way, we have access to the internal network without no foothold in it. 

This wraps up the first part of Exploring Exchange series. In the next parts, the powershell endpoint and OAB will be covered!

[ruler]: https://github.com/sensepost/ruler
[atomizer.py]: https://github.com/byt3bl33d3r/SprayingToolkit/blob/master/atomizer.py
[ptsecurity’s blog]: https://swarm.ptsecurity.com/attacking-ms-exchange-web-interfaces/
[peas]: https://github.com/FSecureLABS/PEAS
[F-SecureLabs]: https://labs.f-secure.com/archive/accessing-internal-fileshares-through-exchange-activesync/
