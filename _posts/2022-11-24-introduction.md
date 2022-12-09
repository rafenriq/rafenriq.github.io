---
layout: page
title: Lab Guide
published: true
---


![](/images/CL-Banner.jpg)


## Introduction

Cisco Catalyst 9166, 9164, and 9162 Series Access Points (AP) are the first AP from Cisco that give you the flexibility to manage your wireless network on premises or in the cloud.
They provide the best of both worlds while delivering flexibility and investment protection for your network.

Cisco understands that selecting your next platform is not an easy choice. With the Cisco Catalyst Wi-Fi 6E access points, you don’t need to make the decision now. Keep the operational mode you use today, whether on premises or cloud management. If your needs change-either way-it’s an easy switch. No new hardware required.

## Lerning Objectives

With this LAB wil walk you through the steps to migrate an AP Wi-Fi 6E from management with 9800 Wireless LAN Controller to Meraki cloud and vice versa and to demonstrate Wi-Fi 6E configuration for both management types. 

## Network Diagram 


![](/images/Picture1.png)


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

Claim your access point via SN, use the SN assigned to your POD

![](/images/createnetwork3.png)





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
