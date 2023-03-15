---
title: "Bypassing Firewalls through proxy servers"
date: 2023-03-14T19:56:46+05:30
draft: false
slug: "bypassing_firewalls_through_proxy_servers"
authors: ["Namami Shanker", "Twisha Bansal"]
tags: ["security", "firewall", "networking"]
categories: ["Networking", "Security", "Proxy Server"]
---

# Introduction

Firewalls are used in various networks to block access to certain content. Examples include college WiFi networks blocking streaming websites like Netflix and game stores like Steam.

We can divert our internet traffic through a proxy server to bypass these restrictions rather than letting it pass directly through the firewall. The firewall will think the internet traffic is going to the proxy server, not the website we are trying to access. Therefore, it won't be able to block the traffic.

This tutorial will show you can bypass the Firewall and send our internet traffic through a proxy server and open up banned website for you. This tutorial especially provides a bang for students' buck because [Azure for Students](https://azure.microsoft.com/en-in/free/students/) provides $100 credits if you login with student ID. You'd be essentially getting this service for free. AWS also has a free tier for 1st year.

# Advantages and Limitations

1. All browser-based traffic will go through the proxy server. This means all the websites banned on the firewall would be **accessible through Mozilla Firefox**. Google Chrome and Microsoft Edge do not allow us to select proxy servers unless you change some system settings (that can be a bit dangerous).
2. To use any peer-to-peer services, such as **torrent** on the network, you must ensure that the server you rented has peer-to-peer connection enabled by the ISP. You can check this by trying to download a torrent using [**transmission-cli**](https://cli-ck.io/transmission-cli-user-guide/) on your remote sever.
3. Installed applications (online games, NetFlix application) directly connect to the internet instead of sending data through the proxy server. If these applications do not provide a setting for selecting a proxy server, their traffic will pass through the firewall.

# Why not a VPN?

Not using a VPN is not a choice but a compulsion. My college WiFi uses FortiNet firewall which has blocked all the major VPNs (including OpenVPN). A proxy server allows me to easily bypass this firewall without the fear that one day this server too will be blocked (they can't find the IP address of my proxy server). 

Also, **as a student you can get small servers for no cost** from Azure or AWS. Azure Students subscription plans gives you $100 credits when you start out. If you manage these credits well you can run your proxy server for a long time (or maybe just create another students account with your roommate's college ID).

# How it works

![Proxy server working](/posts/media/bypassing_firewalls/ProxyServer.svg)

The above diagram shows the simplest example of a Proxy Server. Your requests and internet traffic is sent to a single server which in turn requests the website on your behalf. The website sends the data back to the proxy server which in turn sends it back to you.

We will be using SOCKS5 proxy and send our data through an SSH tunnel. This is safer than HTTP or HTTPS proxy and is quite famous for bypassing firewalls.

# Let's get started with it

## Step 1: Get a server

First of all you need a server you can turn into a Proxy Sever. It can be an already existing server you use for computing or a low cost basic tier server rented from Azure, AWS, Linode or any other Cloud Computing services.

In this example I'll show you how to create a server on Azure but the process is similar for AWS, Linode or any other service. Just make sure you **get a static IP Address**. Follow the short video below to create your own server in a minute.

{{< rawhtml >}}    
<div style="position:relative;width:fit-content;height:fit-content;">
    <a style="position:absolute;top:20px;right:1rem;opacity:0.8;" href="https://clipchamp.com/watch/MvTzp1gTt9Z?utm_source=embed&utm_medium=embed&utm_campaign=watch">
        <img style="height:22px;" src="https://clipchamp.com/e.svg" alt="Server Creation" />
    </a>
    <iframe allow="autoplay;" allowfullscreen style="border:none" src="https://clipchamp.com/watch/MvTzp1gTt9Z/embed" width="640" height="360"></iframe>
</div>
{{< /rawhtml >}}

**Make sure you download the key file.** We will use it later to generate PuTTY's ppk file. You have now rented a server from Azure. Click on the "Connect" button on top and then select "SSH". You will be brought to the following page below:

![Server Connect](/posts/media/bypassing_firewalls/bypassing_firewalls_server_login.png)

Note down you username and IP Address. We will use it now to create an SSH Tunnel.

## Step 2: Create a Dynamic SSH Tunnel to your server

### Windows

1. Download and install [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) on your system.

2. If you remember we downloaded a key file when we created our server. Now search for an installed software "PuTTYgen" and open it. It must have installed along with PuTTY. Load your existing key file.

![Import Key File](/posts/media/bypassing_firewalls/import_pem_key.png)

Now click on "Save private key" and save it in a personal folder like "Documents" etc. Click "Yes" when PuTTYgen asks if you want to save it without a passphrase. We don't need that much security. Just make sure you don't share this private key with anyone else.

2. Open PuTTY and paste your server's **\<username\>@\<ipaddress/domain\>** in "Host Name" text box of the "Session" section of the tree displayed on the left side of the window. It should look like below:-

![PuTTY Host Name](/posts/media/bypassing_firewalls/bypassing_firewalls_PuTTY_hostname.png)

3. Now go to **Connection > SSH > Auth > Credentials** in the left tree. Click on the "Browse" button of "Private key file for authentication" and select the **ppk** key file we generated in the previous step .

![Select PPK key file](/posts/media/bypassing_firewalls/select_ppk.png)

4. Now go to **Connection > SSH > Tunnels** section and enter a "Source Port" of your choice. Make sure this port is free on your system currently, i.e. no application is using this port. Select "Dynamic" radio button in the bottom and click on "Add". This setting will create a dynamic SSH tunnel from your chosen port to the Proxy Server.

![Dynamic tunneling port 6969](/posts/media/bypassing_firewalls/dynamic_tunneling.png)

5. Go back to the Session section. Now we need to save these settings for future use. Type a name of your choice in the "Saved Sessions" text box and click on "Save" button.

![Saving sessions](/posts/media/bypassing_firewalls/saving_sessions.png)

Now the next time you start PuTTY you just need to select your Saved Session and click on the Load button. All the settings done in the above steps will be automatically filled.

6. Cick on the "Open" button to start the SSH session. You should be greeted by a dialog box asking if you should add this server to the list of Known Hosts. Select yes and now you should be in you server displayed by an empty terminal like below. 

![SSH Terminal](/posts/media/bypassing_firewalls/ssh_terminal.png)

As long as this terminal is open and responding, your tunnel is active and you can send internet traffic through it. If you want to close the connection just close this terminal window.

## Linux and MacOS

Creating a dynamic SSH tunnel is rather simple on Linux and MacOS. Just open your terminal and enter the following command:-
```bash
ssh -D <local port> <username>@<ip/domain>
```

Fox example:-

```bash
ssh -D 6969 azureuser@4.240.104.28
```

Just make sure the port you give is free and no application is using it. Avoid ports 1 to 1024 as they are reserved for other applications. The dynamic SSH tunnel has been created and as long you keep the terminal open you can connect to the proxy server.

## Step 3: Download Mozilla Firefox and connect to the server

Mozilla Firefox offers important Proxy setings that chromium based browsers like Google Chrome, Edge, Brave do not provide. [Install Firefox](https://www.mozilla.org/en-US/firefox/new/) and go to its Settings. Scroll to the bottom of "General" settings and click on the "Settings" button next to "Configure how Firefox connects to the internet" of the "Network Settings" section.

![Firefox settings](/posts/media/bypassing_firewalls/firefox_settings_location.png)

This will open the "Connection Settings" dialog box. Select "Manual proxy configuration" and enter "localhost" in the "SOCKS Host" and the tunnel port number in the "Port" section next to it. Also make sure "SOCKS v5" radio button is selected below.

![Firefox settings](/posts/media/bypassing_firewalls/firefox_settings.png)

Click OK and now you are set.

# Enjoy open internet

You can now browse the internet freely without any fear of the firewall through your Firefox browser.

![Free internet](/posts/media/bypassing_firewalls/free_internet.png)

The internet speed will be a bit slower but much faster than other Free VPN alternatives.

# Conclusion

This blog post is for the Firefox as it is the only popular browser that allows us to use SOCKS5 proxy. Google Chrome and Edge would require you to change you System's Network settings which is not flexible. Also Azure's servers do not allow torrent downloads so you can't download torrents with this method. Installed applications and games, for example - Valorant, generally do not provide settings to select a proxy server so the only option is to use a VPN in that case. 

**Remember to stop the server if you are not gonna use it for a long time.**

Anyways, enjoy privacy and freedom. I'll see you in the next one.