---
layout: page
title: Lab Guide
published: true
---


# Contents
 - [Introduction](#specification) 
 - [Learning Objectives](#dependencies-title) 
 - [Network Diagram](#specification) 
 - [Cisco Catalyst 9800 Series Wireless Controller](#cisco-catalyst-9800-series-wireless-controller)
-	[Day 0 configuration](#day-0-configuration)
-	[AP join](#ap-join)
-	[DNA Spaces Integration](#dna-spaces-integration)
-	[Catalyst 9800 Wireless Controllers Configuration Model](#catalyst-9800-wireless-lan-controller-configuration-model)
>    -	[Troubleshooting tools](#troubleshooting-tools)
 - [Meraki Dashboard](#dependencies-title) 
 	- [Mofify your network information](#specification) 
 	- [Configuring Enterprise SSID](#dependencies-title) 
  	- [Configuring Guest SSID](#specification) 
 	- [Meraki Splash page](#splashpage) 
 	- [SSID Availability](#specification) 
 	- [Radio Settings](#dependencies-title) 


## Introduction

Cisco Catalyst 9166, 9164, and 9162 Series Access Points (AP) are the first AP from Cisco that give you the flexibility to manage your wireless network on premises or in the cloud.
They provide the best of both worlds while delivering flexibility and investment protection for your network.

Cisco understands that selecting your next platform is not an easy choice. With the Cisco Catalyst Wi-Fi 6E access points, you don’t need to make the decision now. Keep the operational mode you use today, whether on premises or cloud management. If your needs change-either way-it’s an easy switch. No new hardware required.

## Lerning Objectives

With this LAB wil walk you through the steps to migrate an AP Wi-Fi 6E from management with 9800 Wireless LAN Controller to Meraki cloud and vice versa and to demonstrate Wi-Fi 6E configuration for both management types. 

## Network Diagram 


![](/images/Picture1.png)


## Cisco Catalyst 9800 Series Wireless Controller

The Catalyst 9800 is the new generation Wireless Controller by Cisco that you can deploy either On-Prem or Private/Public cloud. With this type of setup you have various features like telemetry, High Availability, programmability and more.

In this section we will cover the basic setup of a Wireless controller and a 916X Access Point when deployed in Private Cloud Controller.

![](/images/9800-networkdiagram.png)


### Day 0 configuration

Follow the instructions to easily setup the controller from console to operate in a network wireless environment. Note that the day0 configuration is used only for the first time in brand new installations or when controller configuration is reset to factory defaults.

_Procedure_

	1.Access the CLI via the vga/monitor console of ESXi. 

Login the Vsphere Client with following credentials:

**Username:** root
**Password:** C1sco12345

![](images/vsphereclient.jpg)

Once there locate the C9800-CL VM and access to console. 


	2.Terminate the configuration wizard (this wizard it’s not specific for wireless controller)

```
Would you like to enter the initial configuration dialog? [yes/no]: no
Would you like to terminate autoinstall? [yes]:yes
```

	3.Optionally set the hostname:

```
WLC(config)#hostname C9800
```

	4.Enter the config mode and add login credentials using the following command:

```
C9800(config)#username dcloud privilege 15 password dcloud
```

	5.Configure the vlan for wireless management interface.

```
C9800#conf t
C9800(config)#vlan 10
C9800(config-vlan)#name wireless_management
```
	
	6.Configure the SVI for wireless management interface, for example:

```
C9800(config)#int vlan 10
C9800(config-if)#ip address 198.19.10.10 255.255.255.0
C9800(config-if)#no shutdown
```

	7.Configure the interface gigabit 2 as trunk and allow mgmt vlan:

```
C9800(config-if)#interface GigabitEthernet2   
C9800(config-if)#switchport mode trunk
C9800(config-if)#switchport trunk allowed vlan 10
C9800(config-if)#shut
C9800(config-if)#no shut
```

	8.Configure a default route:

```
C9800(config-if)#ip route 0.0.0.0 0.0.0.0 198.19.10.254
```

	9.Disable the wireless network to configure the country code:

```
C9800(config)#ap dot11 5ghz shutdown 
Disabling the 802.11a network may strand mesh APs.
Are you sure you want to continue? (y/n)[y]: y
C9800(config)#ap dot11 24ghz shutdown 
Disabling the 802.11b network may strand mesh APs.
Are you sure you want to continue? (y/n)[y]: y
```

	10.Configure the AP country domain. This configuration is what will trigger the GUI to skip the DAY 0 flow as the C9800 needs a country code to be operational:

```
C9800(config)#ap country US,MX
```

	11.Enable the 802.11a and 802.11b/g networks

```
C9800(config)# no ap dot11 24ghz shutdown
C9800(config)# no ap dot11 5ghz shutdown
```

	12.A certificate is needed for the AP to join the virtual C9800. This can be created automatically via the DAY 0 flow or manually using the following commands.

Specify the interface to be the wireless management interface

```
C9800(config)#wireless management interface vlan 10
```

In exec mode, issue the following command:

```
C9800(config)#wireless config vwlc-ssc key-size 2048 signature-algo sha256 password 0 Cisco123
Configuring vWLC-SSC…
Script is completed
```

This is a script the automates the whole certificate creation.

	13.Verify Certificate Installation:

```
C9800#show wireless management trustpoint
Trustpoint Name : ewlc-default-tp
Certificate Info : Available
Certificate Type : SSC
Certificate Hash : XXXXXXXXX
Private key Info : Available
```

	14.Access via GUI using your credentials, type:

<https://198.19.10.10/>


### AP join


Lately we will see that the Access Points can be mapped to the tags either statically or as part of the rule engine that runs on the controller and comes into effect during the AP join process.

## DNA Spaces Integration

### Catalyst 9800 Wireless LAN Controller Configuration Model

The Catalyst 9800 configuration model was designed to be simple, flexible and reusable. This configuration model takes advantage of the use of profiles that are contained within tags that are eventually applied to the access points. 

![]({{site.baseurl}}/images/config model.png)

In this section we will go through the creation of the creation of the required Tags and Profiles to configure our Access Points to provide service. 


### Troubleshooting tools

### Meraki Dashboard

The first thing that we need to do is to create an account in Meraki dashboard. 

_For this LAB we have created the account, as the linceses available for the APs we are using were given by meraki under the same account._

If you like, you are welcome to create your dashboard account using the following [link](https://account.meraki.com/login/new_account). 

The information needed to create the account is the following: 

![](/images/select-region-dashboard.png)

![](/images/new-dashboarb-account.png)

Credential for the account are: 
username: rafenriq@cisco.com
Password: .......

Once you log in to the Dashboard you will see the following page: 

![](/images/dashboard.png)

Next step is we need to create your own network for that follow the next screen shoots: 

![](/images/createnetwork1.png)

Enter a name for your network

![](/images/createnetwork2.png)

To add the AP to the network we need the serial number (SN). SN can be found on a label in the AP as shown in the following picture. 

![](/images/labelAP.jpeg)

For this LAB, (we are not allowed to bring equipment to the room) the following are the SN, use the SN assigned to your POD

![](/images/createnetwork3.png)

Finally, create the network

![](/images/createnetwork4.png)

In the following page we can place the divice in a specific address , this will define the location for your AP in your network. 

![](/images/adddevicemap1.png)

Select the AP and follow the steps in the image below

![](/images/adddevicemap2.png)

Click on "Enter address" and enter a valid address to position your AP. 

![](/images/adddevicemap3.png)

Enter the address again on the input box in the map, to make sure the AP has been position correctly and "save the device placement"

![](/images/adddevicemap4.png)

Go to the oganization overview, you should see by now your network with the AP added and status green, meaning AP is online and ready to be configured. 

![](/images/justaddednw1.png)

Click on your network name and then on the AP name, at this moment is using AP mac address as name

![](/images/configAP1.png)

Give your AP a name

![](/images/configAP2.png)

Click on BSSID detail and you will see we  already have an SSID configured with name: "network" "WiFi", this is expected as we use the default template, while creating the network. 

![](/images/configAP3.png)

#specification

## Modify your network information 

We can update information to the network just created. In this case let's set the right local time, as the organization was created selecting North America, we need to update this information. Make sure you are under your network before saving changes. Follow the image below.

![](/images/modifynw1.png)

![](/images/modifynw2.png)

Additianlly we can configure SysLog and SNMP for network management. This values will be valid for all the devices you add to this network 

![](/images/modifynw3.png)

![](/images/modifynw4.png)

Finally save the changes

![](/images/modifynw5.png)


## Configuring Enterprise SSID

Navigate to Wireless => SSID. Under "Unconfigured SSID 2" click "edit settings"

![](/images/ssid1.png)

Configure the following parameters:
- SSID name: give it a name Ex. "Corp rafenriq"
- Status: Enable 
- Security: Enterprise with "Meraki Cloud Authentications" 
- Encryption: WPA2
- Client IP and VLAN: "Meraki AP assigned (NAT mode)"
Once done save the changes on the right down corner and navigate back to Wireless => SSID
SSID should look like as following: 

![](/images/ssid2.png)


## Configuring Guest SSID

Navigate to Wireless => SSID. Under "Unconfigured SSID 3" click "edit settings"

Configure the following parameters:
- SSID name: give it a name Ex. "Guest rafenriq"
- Status: Enable 
- Security: Open
- Splash Page: Sign-on with "Meraki Cloud authentication"
- For Advanced splash setting, let's use the following:
..- Captive portal strength: Chose type of traffic allowed before completing portal
..- Walled garden: IP addresses that can be allowed before going through the splash page, for example if portal is hosted in a 3rd party server, we need to allow this IP address so clients can reach to this portal
..- Self-registration: Display a form on the web page to allow client to create a user, users need to be Admin approved before they can actually work in the portal.  
..- Simultaneous logins: allow to use same user/password in different devices
..- Controller disconnection behavior: If AP is not reaching Meraki Dashboard, what is the expected behavior. 


![](/images/ssid3.png)

- Client IP and VLAN: "Meraki AP assigned (NAT mode)"
Once done save the changes on the right down corner and navigate back to Wireless => SSID
SSID should look like as following:

![](/images/ssid4.png)

#splashpage
## Meraki Splash page

Unlike Splash page presented by 9800, Meraki has a very good user friendly interface on the dashboard to modify the default splash page to customize your own. 
All we need is a bit of HTML knownledge. 

In order to modify the splash page navigate to: Wireless >> Splash page
Select the SSID we just created for guest clients. 
From there we have available 2 themes, wich we can only modify colors as shown below, click on "Preview" to check how your new colour selections looks like. Feel free to play around with it. 

If we want to further customize this, using the themes as a template, we can do the following: 

![](/images/splashpage1.png)

![](/images/splashpage2.png)

![](/images/splashpage3.png)

![](/images/splashpage4.png)

You will have a new tab on your browser to be able to modify the theme as needed. 
- Edit: will take us to the html code to be edited 
- Pop up: it is a preview of the changes made. 

![](/images/splashpage5.png)

In this case we are doing something very simple, we are just adding a the following message to the page: "This message has been added for CL LAB LTREWN-2034" and we are changing the color for the font. 

Select "auth.html" and click Edit, add the following line to the html file and "Save" the file 

![](/images/splashpage8.png)

Lets select file "main.css" to configure the color and lets make the following change

![](/images/splashpage9.png)

After you save the changes you should see them as following, You can play around addind more stuff if you want to. 
(the following link has the hex value for the colors you can use on the .css file [color codes](https://www.quackit.com/css/css_color_codes.cfm)) 

![](/images/splashpage10.png)

Click on "back to config" and select your new custom theme, so user will be seeing this page. On this page you can still configure the following if you like: welcome message, logo, and how frequent you want users to see the portal when they connect to the wireless network.
We have done the following changes to show the configuration 

![](/images/splashpage11.png)

"Save the changes". Our final page looks like: 

![](/images/splashpage12.png)

#radiosettings
### Radio Settings

Now let's modify the radio settings for our AP. Navigate to "Wireless" >> "Radio Settings". From there you can see our AP is using the "Basic Indoor Profile", which is the defaul added when creating the network, but most likely in a real deployment we do not want to use this one. As wireless networks changes based on where AP are deploy. You should see something silar as shown in the next image:




















<dl>
  <dt>Definition list</dt>
  <dd>Is something people use sometimes.</dd>

  <dt>Markdown in HTML</dt>
  <dd>Does *not* work **very** well. Use HTML <em>tags</em>.</dd>
</dl>


As opposed to traditional hardware that requires physical access to be configured, you can configure everything before you even have your devices
Once your device reaches Meraki cloud it will be able to download its configuration and report its performance. 

Here the list of things we need to setup: 
-Create an organization and account 
-Serial number of your device 
-Uplink to internet so they can talk to Meraki cloud


Markdown | Less | Pretty
--- | --- | ---
*Still* | `renders` | **nicely**
1 | 2 | 3



Colons can be used to align columns.

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |


Testing one more table. 

| Column 1 Header | Column 2 Header | Column 3 Header |
| --------------- | --------------- | --------------- |
| Row 1 Column 1 | Row 1 Column 2 | Row 1 Column 3 |
| Row 2 Column 1 | Row 2 Column 2 | Row 2 Column 3 |
| Row 3 Column 1 | Row 3 Column 2 | Row 3 Column 3 |
