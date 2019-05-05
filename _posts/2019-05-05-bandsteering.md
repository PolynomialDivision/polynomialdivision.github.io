---
layout: post
title: Bandsteering in OpenWrt
categories: openwrt
---

People are interested in bringing band-steering to OpenWrt. There is even a [hostapd patch](https://lists.01.org/pipermail/connman/2016-March/000496.html) supporting band-steering. This patch can not directly be used on OpenWrt because OpenWrt opens two separate hostapd instances for both radios.
That is why I present other approaches.

## 802.11k,v amendments

For bandsteering 802.11k for asking the client what is in his range and send some 802.11v fast bss transition to the client to switch to 5Ghz. Then 802.11r would work too.

There are different milestones that have to be accomplished:

- [Network-Manager Patch](https://gitlab.freedesktop.org/NetworkManager/NetworkManager/merge_requests/6) has to be accepted for supporting 802.11r (if u use network-manager...)
- Fast BSS Transition be reachable via ubus

When some daemon could be implemented independent of the hostapd.

## DAWN

DAWN supports band-steering even across different devices. Just setting this [weigth](https://github.com/berlin-open-wireless-lab/DAWN/blob/master/files/dawn.config#L36). Still DAWN has serious limitations that are caused by the up-to-dateness of the hearing map!