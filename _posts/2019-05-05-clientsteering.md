---
layout: post
title: Client Steering with legacy Devices
categories: openwrt
---

WiFi introduced `802.11k` and `802.11v`. This new amendments introduced new Radio Resource Management (RRM) opportunities.  Still, there are a lot of legacy devices in the wild not supporting those amendments. Especially, you find those devices in the `2.4 GHz ISM` band that is highly crowded. In this blog article I will subscribe how to handle with those legacy devices.

### Association Control

|WiFi Association|
|---|
| ![Association Control](/static/img/assocdeassoc.png)| 

#### Scanning

Typically, a client devices scans for nearby APs by sending out probe request messages. The AP will respond with probe response messages containing information about supported rates, network capabilities and further information. That process is called `active scanning`. Furthermore, the client has the ability to collect `beacon frames` to learn abouth other APs in its range. This process is called `passive scanning`.
Typically, the client connects to the AP with the highest signal strength.

#### Authentication
The authentication is before the association. Not that important to explain.
The authentication can be declined by the AP using a status code which include the reason.

#### Association
In the association the clients associates with the AP. After a successful association it is able to exchange data with the network.
Like the authentication, the association can be declined by the AP using a status code. A huge adverse is that declining the association needs a successful authentication beforehand. So the client needs to exchange at least two messages (often 4 if the client exchanges probe messages).
 
#### De-Association
The de-association can be triggered by the AP or the client itself. Both are able to give a reason code for that.

### Associtation Control
Association control has some difficulties. There are different ways to steer a client to the right AP. All the control mechanisms come with advantages and disadvantages. At the end, a combination should be used to implement a feasible solution.

#### Controlling the Probe Exchange

|Scanning|Answering|
|---|---|
|![Associtation Control](/static/img/assoc_1.png)|![Association Control](/static/img/assoc_2.png)|

A very efficient control is to suppress probe response message. If the client does not receive a probe response, it will not start an association to the AP.  Therefore, no further message overhead is created. At the end, the chosen AP is the only one, responding to the client with a probe response. Sadly, suppressing the probe response does not work all the time. The client can learn about the AP through passive scanning, because It is not possible to suppress beacon frames. Beacon frames are essentially for maintaining the WiFi network.
In addition, the client can directly try to associate to an already known AP without exchanging probe requests.

#### Declining Association

|Try Associtation|Decline Associtation|
|---|---|
|![Associtation Control](/static/img/deny_assoc_1.png)|![Association Control](/static/img/deny_assoc_2.png)|

If the controlling the probe exchange is not successful the AP can further deny the authentication or association. Denying the authentication is faster then declining the association.

|Status Code|Description|
|---|---|
|...|...|
|17|Association denied because the AP is unable to handle additional stations.|
|...|...|

Looking at the IEEE 802.11 standard, the are only a few interesting status code that can be used. Still, the status codes are often related to the association and not the authentication. Of course, you can use the same status code for declining the authentication. It depends on the driver implementation, how the client will handle that.

#### Deauthentication

|Try Associtation|Decline Associtation|
|---|---|
|![Deauthentication Control](/static/img/dissassociation_1.png)|![Deauthentication Control](/static/img/dissassociation_2.png)|

If the connection of a link decreases or the situation is changing, the controller might want to sihft the station to another AP. For that it can deassociate the client. (better alternative would be `802.11v`)

Reason Codes|Description|
|---|---|
|...|...|
|5|Disassociated because AP is unable to handle all currently associated stations.|
|...|...|

After the deassociation of the client, the previous mechanism can be used to guide the client to the right AP.

##### Other Handover Techniques: Channel Switch Announcement
There are techniques to force a client to another AP by sending a channel switch announcement to the client without actually switching the channel. For example there are two APs: On at 2.4 GHz and another at 5 GHz. Both run with the same BSSID. Now the 2.4 GHz AP would send some channel switch announcement to the client that it now switches to the 5 GHz frequency. However, the interface won't switch the frequency but the client will switch. However, the interface won't switch the frequency but the client will switch to the new interface which has the same BSSID. This is some kind of seamless handover.

### Hearing Map

There are different ways to create a hearing map. I just describe one way of doing it.

#### Hearing Map based on Probe Exchange

The probe request and response exchange has a great impact with which
AP the station will associate. In this step the client will learn about the APs it can connect to. The
other way around an AP only takes notice of the station if it sends a probe request. Different wireless
devices treat this process differently. Not every wireless device is scanning every channel and then
decides to which AP to connect. It is necessary for a hearing map that the station sends a probe request to all APs that are in its range. The probe requests allow to create a hearing map for the station and gather important information like the RSSI, transmission rates, capabilities and the frequency.
That is why the laptop has to be forced by the APs to scan every channel. So first probe response message and association tries can be declined. Only then it is allowed to join an access point. All the probe
requests will be collected. At the end, this allows to get information about the APs (BSSIDs) the station
can connect to.
**Limitations:** This only works in static scenarios. The up-to-dateness depends on the clients background scanning frequency. Scanning costs energy. Therefore, mobile devices scan as less as possible.
Furthermore, some devices have mac-randomization for their probe requests. Therefore, probe requests might not be assignable to the actual devices.

#### Other Hearing Map Approaches

If an AP has two radios that can be controlled separately, one radio can hop periodically over all other channels. On those channels it can receive and decode the frames and use them to get signal strength information to other clients. In addition, APs can send data null frames to clients to create traffic the other stations can hear.

If an AP has only one radio, it can still hop through all other channels. Of course, in the timegap the AP is hopping through all channels, no transmission can happen with the AP. Further, the AP has to signal this to its clients.