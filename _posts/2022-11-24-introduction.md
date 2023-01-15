---
layout: page
title: Lab Guide
published: true
---


# Contents
 - [Introduction](#specification) 
 - [Learning Objectives](#dependencies-title) 
 - [Network Diagram](#specification) 
 - [Get Started](#get-started)  
 - [Cisco Catalyst 9800 Series Wireless Controller](#cisco-catalyst-9800-series-wireless-controller)
	-	[Day 0 configuration](#day-0-configuration)
	-	[AP join](#ap-join)
	-	[Catalyst 9800 Wireless Controllers Configuration Model](#catalyst-9800-wireless-lan-controller-configuration-model)
	-	[Troubleshooting tools](#troubleshooting-tools)
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
This lab guide also provides some troubleshooting tools from both worlds.

## Network Diagram 


![](/images/Picture1.png)

## Get Started

To access your session

* Open Anyconnect, add your credentials and click "Connect". 

**Host:** dcloud-rtp-anyconnect.cisco.com

Select from the table your corresponding credentials:

| **Pod**         | **User**          | **Password**  |
| ------------- |:-------------:| ---------:|
| 1     | `v2718user1`       | `3b48dc`    |
| 2     | ``       | ``    |
| 3     | ``       | ``    |
| 4     | ``       | ``    |
| 5     | ``       | ``    |
| 6     | ``       | ``    |
| 7     | ``       | ``    |
| 8     | ``       | ``    |
| 9     | ``       | ``    |
| 10     | ``       | ``   |
| 11     | ``       | ``    |
| 12     | ``       | ``    |
| 13     | ``       | ``    |
| 14     | ``       | ``    |
| 15     | ``       | ``    |


* Once VPN connection is stablished open a Remote Desktop Connection

![](/images/rdp.png)

**Host:** 198.18.133.136

**User:** Administrator

**Password:** C1sco12345


## Cisco Catalyst 9800 Series Wireless Controller

The Catalyst 9800 is the new generation Wireless Controller by Cisco that you can deploy either On-Prem or Private/Public cloud. With this type of setup you have various features like telemetry, High Availability, programmability and more.

In this section we will cover the basic setup of a Wireless controller and a 916X Access Point when deployed in Private Cloud Controller.

![](/images/9800-networkdiagram.png)


### Day 0 configuration

Follow the instructions to easily setup the Wireless Controller. Note that the day0 configuration is used only for the first time in brand new installations or when controller configuration is reset to factory defaults. You can choose between GUI or CLI

**_Web UI Procedure_**

1.Access the Day 0 Wizard via Out-Of-Band port. 

Open a browser and type **https://100.64.0.7** or click on the **C9800-CL** bookmark. Use following credentials.

**Username:** dcloud 

**Password:** dcloud 

![](/images/GUI-credentials.png)

2. Once you are logged into the controller, fill in the following:

In the General Settings screen, 

• Deployment mode – Standalone
• Country Code - US,MX,BE
• NTP Server - 100.64.0.1

![](/images/Day0_1.png)

• Wireless Management Settings

  o Port number - GigabitEthernet2
  
  o VLAN - 10
  
  o IPv4 - Check
  
  o Wireless Management IP - 198.19.10.7
  
  o Subnet mask - 255.255.255.0
  
• Static Route Settings

  o IPv4 Route - Check
  
  o IPv4 Destination Prefix - 0.0.0.0
  
  o IPv4 Destination Mask - 0.0.0.0
  
  o IPv4 Next Hop IP - 198.19.10.254
  
  Click "Next"

![](/images/Day0_2.png)

In the Wireles Network Settings >+Add,

• Add Network

  o Network Name - CiscoLive
  
  o Network Tye - Employee
  
  o Security - WPA2 Personal
  
  o Pre-Shared Key - Cisco123
  
  Click "+Add" and "Next"

![](/images/Day0_3.png)

In the Advanced Settings,

• AP Certificate

  o Generate Certificate - Yes
  
  o RSA Key-Size - 2048
  
  o Signature Algorithm - sha256
  
  o Password - C1sco12345
  
• Create a New AP Management User

  o New AP Management User - admin
  
  o Password - C1sco12345
  
  o Secret - C1sco12345
  
  Click "Summary"

![](/images/Day0_4.png)

![](/images/Day0_5.png)

If all settings are correct click "Finish" and proceed.

![](/images/Day0_6.png)

![](/images/Day0_6_1.png)

3.After a minute or two refresh the page and add the credentials. 
Now you will be prompted with the Wireless Controller GUI. 


**_CLI Procedure_**

1. Access to controller via SSH.

Access the controller CLI via SSH using mRemoteNG App. Once there go to connections and click on **C9800**. 

Login with the following credentials:

Username: dcloud 

Password: dcloud 

Enable: dcloud

![](/images/mremoteng-ssh.png)


2.Set the hostname as seen below:

```
WLC#conf t
WLC(config)#hostname C9800
```

3.Configure vlan 10 for the wireless management.

```
C9800#conf t
C9800(config)#vlan 10
C9800(config-vlan)#name wireless_management
```
	
4.Configure the SVI for Wireless Management Interface (WMI) as follows:

```
C9800(config)#int vlan 10
C9800(config-if)#ip address 198.19.10.7 255.255.255.0
C9800(config-if)#no shutdown
```

5.Configure the GigabitEthernet 2 as trunk and allow Vlan 10:

```
C9800(config-if)#interface GigabitEthernet2   
C9800(config-if)#switchport mode trunk
C9800(config-if)#switchport trunk allowed vlan 10
C9800(config-if)#shut
C9800(config-if)#no shut
```

6.Configure the WMI default route:

```
C9800(config-if)#ip route 0.0.0.0 0.0.0.0 198.19.10.254
```

7.Disable the wireless network to configure the country code:

```
C9800(config)#ap dot11 5ghz shutdown 
Disabling the 802.11a network may strand mesh APs.
Are you sure you want to continue? (y/n)[y]: y
C9800(config)#ap dot11 24ghz shutdown 
Disabling the 802.11b network may strand mesh APs.
Are you sure you want to continue? (y/n)[y]: y
```

8.Configure the AP country domain. This configuration is what will trigger the GUI to skip the DAY 0 flow as the C9800 needs a country code to be operational:

```
C9800(config)#wireless country BE
C9800(config)#wireless country MX
C9800(config)#wireless country US

```

9.Enable the 802.11a and 802.11b/g networks

```
C9800(config)# no ap dot11 24ghz shutdown
C9800(config)# no ap dot11 5ghz shutdown
```

10.Specify the interface, in this case Vlan 10, to be the Wireless Management Interface.


```
C9800(config)# wireless management interface Vlan10
C9800(config-mgmt-interface)#exit
```

11.A certificate is needed for the AP to join the virtual C9800. This can be created automatically using the following commands.

```
C9800(config)#wireless config vwlc-ssc key-size 2048 signature-algo sha256 password 0 Cisco123
Configuring vWLC-SSC…
Script is completed
```

> This is a script that automates the whole certificate creation.

12.Verify Certificate Installation:

```
C9800#show wireless management trustpoint
Trustpoint Name  : C9800_WLC_TP
Certificate Info : Available
Certificate Type : SSC
Certificate Hash : 31ee336150e5e2b2ee1f8e554c20dabac6ecd6af
Private key Info : Available
FIPS suitability : Not Applicable
```

14.Access via GUI using your credentials.

Open a browser and type **https://100.64.0.7** or click on the **C9800-CL** bookmark.

**Username:** dcloud 

**Password:** dcloud 

![](/images/GUI-credentials.png)

### AP join

For this lab configure the WLC Public IP address so the APs can reach the controller through the internet. 

| **Pod**         | **Public IP**     |
| ------------- |:-------------:|
| 1     | `64.100.10.X`       |
| 2     | `64.100.10.X`       |
| 3     | `64.100.10.X`       | 
| 4     | `64.100.10.X`       |
| 5     | `64.100.10.X`       |
| 6     | `64.100.10.X`       | 
| 7     | `64.100.10.X`       | 
| 8     | `64.100.10.X`       | 
| 9     | `64.100.10.X`       | 
| 10     | `64.100.10.X`       | 
| 11     | `64.100.10.X`       | 
| 12     | `64.100.10.X`       | 
| 13     | `64.100.10.X`       |
| 14     | `64.100.10.X`       |
| 15     | `64.100.10.X`       |

Select your corresponding Public IP address then go to **Configuration > Interface > Wireless**, click ont he Management interface and add it to the NAT IPv4 field as in below image.

![](/images/wmi-nat.png)


Once the Public IP is configured you will see your assigned Access Point joining to your brand new 9800 controller. 


Verify your AP joined using "**show ap summary**" command

```
C9800#show ap summary 
Number of APs: 1

CC = Country Code
RD = Regulatory Domain

AP Name                          Slots AP Model             Ethernet MAC   Radio MAC      CC   RD   IP Address                                State        Location
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
CW9166I-A-6                      3     CW9166I-A            cc9c.3ef7.e440 6c8d.772e.63a0 MX   -A   172.16.26.189                             Registered   default location    
```

The way Access Points work in the 9800 is by usings tags, they are used to control the features that are available for each AP. The tags are assigned to the access points as part of the rule engine that runs on the controller and comes into effect during the AP join process.

Verify what tags where applied to the access point. Use "**show ap tag summary**" command. 

```
C9800#show ap tag summary 
Number of APs: 1

AP Name                           AP Mac           Site Tag Name                     Policy Tag Name                   RF Tag Name                       Misconfigured    Tag Source    
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
CW9166I-A-6                       cc9c.3ef7.e440   default-site-tag                  default-policy-tag                default-rf-tag                    No               Default       
```

If you prefer GUI, you can verify if AP is joined from "**Configuration > Wireless > Access Points >**"

![](/images/AP_joined.png)

To check the current tags click on the AP entry

![](/images/AP_tags_default.png) 

As you can observe when a brand new Access Point joins, the controller assings the default tags. 




### Catalyst 9800 Wireless LAN Controller Configuration Model

The Catalyst 9800 configuration model was designed to be simple, flexible and reusable. This configuration model takes advantage of the use of profiles that are contained within tags that are eventually applied to the access points. 

![](/images/config model.png)

There are three tags:

- **Policy Tag**. Link between a WLAN Profile (SSID) and a Policy Profile
- **Site Tag**. Defines de AP mode and other AP settings, trough AP join profile and Flex profile.
- **RF Tag**. Sets the RF profiles with the seetings for each band.

Let's create the Profiles and Tags with basic configurations. Later we will apply these tags to the Access Point joined to the controller.

_Procedure:_

1. Access the controller GUI with your credentials.

**GUI:** https://100.64.0.7

**Username:** dcloud

**Password:** dcloud
    
<!-- ![](/images/gui_log_in.png) -->

Note: To configure Tags and Profiles we will use the **Configuration > Tags & Profiles** section,

![](/images/Tags_and_profiles.png)

2.Create a new SSID. 

Go to **Configuration > Tags & Profiles > WLAN > +Add**

![](/images/wlanprofile_add.png)

WLAN creation window will pop up, give it a name and enable it. 
Note: For this excercise we will disable 6GHz band.

![](/images/wlanprofile_general.png)

Go to the Security tab, set configurations as in the image below.

![](/images/wlanprofile_security.png)

This is how your new WLAN looks like.

![](/images/WLAN.png)

3.Create a new Policy Profile. 

Go to **Configuration > Tags & Profiles > Policy > +Add**

![](/images/Policyprofile_add.png)

Give it a name and enable it. 

![](/images/Policyprofile_general.png)

Go to the Access Policies tab. Here you can define the Vlan name or Vlan ID for the SSID, in this case we will use the Vlan name "Clients" that will be later defined in the Flex Profile.

![](/images/Policyprofile_access.png)

This is how your Policy Profile looks like.

![](/images/PolicyProfile.png)

4.Create the Policy tag and map the created WLAN and Policy Profiles.

Go to **Configuration > Tags & Profiles > Tags > Policy > +Add**

![](/images/tag_policy_AMS.png)

5.Create an AP join Profile

Go to **Configuration > Tags & Profiles > AP join > +Add**

![](/images/JoinProfile_add.png)

6.Create a Flex Profile. 

Go to **Configuration > Tags & Profiles > Flex > +Add**. Give it a name and define the AP native vlan, in this case we will use Vlan ID 30. This is the same value as the native vlan configured in the AP switchport.

![](/images/FlexProfile_add_General.png)

Select the VLAN tab, set configurations as in the image below.

![](/images/FlexProfile_vlan.png) 

7.Create a new Site tag and apply the created AP join and Flex Profiles.

Go to **Configuration > Tags & Profiles > Tags > Site > +Add**

![](/images/tag_site_AMS.png)


### Troubleshooting tools

To troubleshoot in the 9800 we have some avaialble tools.

### Syslog
Show the logs generated by the 9800 WLC, this output shows general logs as well as some wireless-specifics logs. 

You can find them at **Troubleshooting > Syslog**
 add image

From CLI:

```
C9800#show logging
....
Log Buffer (131072 bytes):
Jan 14 21:33:50.153: %PKI-6-TRUSTPOINT_CREATE: Trustpoint: C9800_WLC_TP created succesfully
Jan 14 21:33:52.186: %PKI-6-CERT_ENROLL_MANUAL: Manual enrollment for trustpoint C9800_WLC_TP
Jan 14 21:33:52.585: %PKI-6-CSR_FINGERPRINT:
                      CSR Fingerprint MD5 : 027FB457F1BFE59767F87641D310B572
                      CSR Fingerprint SHA1: ABA9024717A535EA77DE79BC920B37106C259AF6
                      CSR Fingerprint SHA2: D5621FDD60523F781E604831FFF2126EE4CA094FFCD79060A1F12A2E1353F2A8
Jan 14 21:33:52.585: %PKI-6-CSR_FINGERPRINT_UNSIGNED:
                      CSR Fingerprint MD5 (unsigned) : 74E9F6F75BDADE5DB03B1192E2E8AFA3
Jan 14 21:33:52.585: CRYPTO_PKI:  Certificate Request Fingerprint MD5 :027FB457 F1BFE597 67F87641 D310B572
Jan 14 21:33:52.586: CRYPTO_PKI:  Certificate Request Fingerprint SHA1 :ABA90247 17A535EA 77DE79BC 920B3710 6C259AF6
Jan 14 21:33:52.586: CRYPTO_PKI:  Certificate Request Fingerprint SHA2 :D5621FDD 60523F78 1E604831 FFF2126E E4CA094F FCD79060 A1F12A2E 1353F2A8
Jan 14 21:33:52.586: CRYPTO_PKI:  Certificate Request Fingerprint MD5  (unsigned):74E9F6F7 5BDADE5D B03B1192 E2E8AFA3
Jan 14 21:33:52.712: %SYS-5-CONFIG_I: Configured from console by  on vty2 (EEM:Mandatory.crypto_pki_vwlc_ssc_config)
Jan 14 21:33:52.984: %PKI-6-CERT_INSTALL: An ID certificate has been installed under
                      Trustpoint   : C9800_WLC_TP
                      Issuer-name  : o=Cisco Virtual Wireless LAN Controller,cn=CA-vWLC_C9800
                      Subject-name : serialNumber=920XHRC7PLW+hostname=C9800,o=Cisco Virtual Wireless LAN Controller,cn=C9800_WLC_TP
                      Serial-number: 02
                      End-date     : 2033-01-13T21:33:47Z

```

### RadioActive tracing

RadioActive traces give the ability to conditionally print debug information across processes, threads for the condition of interest. The most used case is when torubleshooting client connectivity,here the conditional debug runs for client mac or ip address to get end to end view at control plane.

add images

Later you can upload the RadioActive traces into the [Wireless Debug Analyzer]https://cway.cisco.com/wireless-debug-analyzer/), this tool parsed debug files to make easier to troubleshoot wireless issues such as client association, authentication, roaming, and connectivity issues. 



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

Additioanlly we can configure SysLog and SNMP for network management. This values will be valid for all the devices you add to this network 

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
