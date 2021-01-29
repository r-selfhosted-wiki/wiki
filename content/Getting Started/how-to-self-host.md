---
Title: How to Self Host
---

Here we will go over the basics of what self-hosting entails. There will be a lack of detail in each individual step as those details are reserved for their own documentation in a separate article or are specific to a provider/service as not everything can be self hosted, especially if you are wanting to make a service publicly available.

### Contents

- [Domain name registration]({{< relref "#registering-a-domain-name">}})
- [DNS configuration]({{< relref "#domain-name-records">}} "Domain Name Records")
- [Network configuration]({{< relref "#network-configuration">}})
- [Server setup]({{< relref "#server-configuration">}} "Server Confirguration")

#### Registering a domain name

Registering a [domain name]({{< ref "explanations/_index.md#domain-names">}} "domain name") is usually the first step when you are looking to have something self-hosted that is reachable via the internet. You will have to find a domain registrar you can trust; recommending specific providers is outside the scope of this wiki.

Top level domain names vary in price. Dot.com domain names are usually ~$15/yr, while others like dot.net or dot.info are typically less. Some can be more expensive, and the registrar will have their price listings based on the top level domain. Domain registration is a competitive business, with lots of resellers working for white-label registrars. You can look at several registrars to compare prices for the domain you want to register.

Domain registrations are on a first-come, first-serve basis. If the domain name you want is already taken, you will have to wait until it expires, or buy it from the current owner. Otherwise, you can come up with a different domain name to register.

**BEWARE**: Some domain registrars will take domain names that you search for, but not purchase, and "hold" their registration.

EG: You go to sumregistrar.com and search for ilikeponies.com to see if it's available for registration. You find that it is available, but want to check out sumotherregistrar.com to see their prices for the same domain. When you check there, the domain ilikeponies.com is taken. But when you search for it again on sumregistrar.com, it's still available.

What is happening is that sumregistrar.com has "held" the registration for a period of time to prevent you from registering it elsewhere in hopes that you take your business with them.

While we are not certain of the legality of such a practice, it is considered uncouth and frowned upon in the industry. You can resolve this by contacting the registrar, telling them they are massive jerks, and have them release the hold.

If they have no shame, you may have to explain that your poor experience with them can wind up on webhostingtalk.com for the world to see.

**[To the top]({{< relref "#contents">}} "Top")**

#### Domain name records

Once you have registered a domain name, you will need to configure the nameservers and then add [DNS records]({{< ref "explanations/_index.md#domain-name-system">}} "DNS records") for the domain that point to the IP address of your server.

Your domain registrar makes use of the nameserver records, and your DNS provider will take care of the other types of records.

The basic DNS records to add are an A record for yourdomainname.com, and a CNAME for www.yourdomainname.com.

Get the IP address of the machine you will use. If you will serve locally, then the local IP address of the machine will not work (EG:192.168.1.X). You will have to use your public IP and use a reverse proxy + VPN or port forward. You can find your public IP address in your router status page or by going to https://ifconfig.me/.

Use that IP as the content for you DNS A record.

See your router documentation for directions on setting up port forwarding to the machine's local IP address.

**[To the top]({{< relref "#contents">}} "Top")**

#### Network Configuration

Your ISP is likely blocking common ports used for internet traffic such as 80 (http), 443 (https), 21 (ftp), 25 (email SMTP), and many others.

Some ISPs will have no problem unblocking some of these ports, but don't expect them to cooperate.

Regardless, there are options to get around these issues if your ISP is non cooperative. There may be clauses in your service agreement stating that hosting services from your home internet connection is prohibited. While ISPs rarely take action against customers if they are found in violation of this rule, there are legitimate reasons for having such a clause.

Generally if you bypass their restrictions to self-host anyway, they don't care unless you cause problems for them.

If you have a business internet plan, these issues don't exist. You may still have to request a static IP, but your ISP should not be blocking ports by default as it is assumed you will be handling that yourself. If they are, they should cooperate with any unblocking requests.

They typical way to bypass these restrictions is either to port forward ( if your ISP doesnt block the ports) or using a reverse proxy in conjunction with a VPN using an internet reachable machine.

You also have to contend with dynamic IP addresses. Your home connection is assigned a public IP address, but it is not fixed, and can change if your DHCP lease on their network expires. Some ISPs may offer to provide you a static IP address for a monthly fee.

For dynamic IP addresses, your DNS provider may support dynamic DNS updates. This can either require a dedicated client, or to simply call a URL with an embedded token with a program like wget or curl. It is an effective way to have a DNS record automatically updated on a regular basis to counteract a changing IP address.

**[To the top]({{< relref "#contents">}} "Top")**

#### Server configuration

You need a server to handle requests for your domain name. This can be a shared hosting account with a webhosting provider, a dedicated server, a virtual private server, or your own server running locally.

This is a self-hosting guide, the assumption that the server is local has been made.

Once you have chosen and installed the operating system you are going to use, you will need to setup your service daemon. For the sake of this article, we will assume that you are wanting to serve a website using your domain name.

There are a variety of webservers available, and the specific details on configuring that webserver will be covered elsewhere.
The basic steps are:

- install the webserver
- configure it to answer for your domain name
- give it a directory to serve from (/some/directory/domainname.com/html)
- upload content into that folder

 Make sure the content folder has the permissions needed by your webserver, and you should be good to go.

**[To the top]({{< relref "#contents">}} "Top")**
