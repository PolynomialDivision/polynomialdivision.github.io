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

Typically, a client devices scans for nearby APs by sending out probe request messages. The AP will respond with probe response messages containing information about supported rates, network capabilities and further information. That process is called `active scanning`. Furthermore, the client has the ability to collect `beacon frames` to find out other stations. This process is called `passive scanning`.
Typically, the client connects to the AP with the highest signal strength.

#### Authentication
The authentication is before the association. Not that important to explain.
The authentication can be declined by the AP using a status code which include the reason.

#### Association
In the association the clients associates with the AP. After a successful association it is able to exchange data with the network.
Like the authentication, the association can be declined by the AP using a status code. A huge adverse is that declining the association needs a successful authentication beforehand. So the client needs to exchange at least two messages (often 4 if the client exchanges probe messages).
 
#### De-Association
The association can be triggered by the AP or the client itself. Both are able to give a reason code for that.

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