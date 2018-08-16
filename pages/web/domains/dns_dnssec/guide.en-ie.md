---
title: Secure your domain with DNSSEC
excerpt: ''
slug: secure_your_domain_with_dnssec
legacy_guide_number: g609
section: Security
order: 1
---


## Prerequisites

- Your domain must be registered with OVH, a technical requirement due to the need for DNS records to be up-to-date on the registry.
- It must contain one of the following extensions: .co.uk, .com, .net, .eu, .fr, .be .pl, .re, .pm, .yt, .wf, .tf, .info, .li, .ch, .biz, .de, .sx, .org, .se, .nl, .in, .us, .at, .nu, .la, .ac, .cz, .me, .sh, .io, .uk, .me.uk, .org.uk, or any other recently-released new extension, such as .london, .club, .xyz, .wiki, .ink, and all other Donuts extensions, (other extensions coming soon).




## Two use cases

- Either you host your domain on third-party DNS servers (your dedicated server, or any other machine).

- Or you use the OVH shared DNS, which is the case study covered by this guide. Then we manage the keys, their periodic rotation, the update of the DS record and the zone signature in a transparent way for you.


If you are unsure of which example applies to you, you can check in your [manager](https://www.ovh.com/manager/web) by selecting your domain and then going to "DNS management". If the listed DNS servers are displayed as follows: nsXX.ovh.net and dnsXX.ovh.net, or Xns200.anycast.me then you are on the OVH DNS servers.


## Activation

- First of all you have to sign in to your 
[control panel](https://www.ovh.com/manager/web).


Then select the relevant domain name in the  "Domains" section.

![](images/img_2896.jpg){.thumbnail}

- You can make sure that you are using OVH dns servers in the "DNS management" section.



![](images/img_3966.jpg){.thumbnail}

- Once you have checked the DNS, click on the Secured Delegation (DNSSEC) button.



![](images/img_3967.jpg){.thumbnail}

- A pop-up window will appear so that you can confirm the operation. You will be told that it may take around 24 hours to activate DNSSEC.



![](images/img_2895.jpg){.thumbnail}

- The secured delegation button will then become animated.



![](images/img_3968.jpg){.thumbnail}

- You can then check in the "operations in progress" tab to see if the operation has begun.



![](images/img_3969.jpg){.thumbnail}


## Deactivation
To disable DNSSEC, select your domain again and then click on the  button to disable DNSSEC. A new pop-up window will appear so that you can confirm the operation. Please note that if activation is in progress, you must wait until it completes (the button will be grayed out), in order not to allow the DNSSEC configuration of your domain in an intermediate state.

![Deactivation](images/img_3970.jpg){.thumbnail}


## Method 1: By using Firefox or Chrome
You can install a Firefox extension that allows to check whether the websites you visit are secured by DNSSEC, and if so, what is the result of the validation. This extension is [available here](http://www.dnssec-validator.cz/). Once installed, you will see a key to the left of the address bar of your browser. For domains where the key is green, the website IP has been verified by DNSSEC.

![Firefox plugin for DNSSEC validation: This website is secure](images/img_119.jpg){.thumbnail}
If the key is orange it means that the recursive DNS of your ISP does not support DNSSEC. Nevermind: you can use alternate DNS servers to perform this validation. Firefox module offers a list, which you can access by right-clicking the key and then selecting "Preferences".

An alpha version of this extension is also available for Chrome on [url=https://chrome.google.com/webstore/detail/hpmbmjbcmglolhjdcbicfdhmgmcoeknm]this page[/ url].


## Method 2: In console mode, with prior notification of the root key
To verify that DNSSEC is correctly configured for a domain, you can use the dig tool. To validate DNSSEC, it needs to know the public root key (with which is signed the key which signed the root zone"."). This key is available in several places on the Internet. To simplify, we reproduce it below, you can copy it as it is in the file /etc/trusted-key.key (everything must be on the same line):


```
. 172717 IN DNSKEY 257 3 8 AwEAAagAIKlVZrpC6Ia7gEzahOR+9W29euxhJhVVLOyQbSEW0O8gcCjF
FVQUTf6v58fLjwBd0YI0EzrAcQqBGCzh/RStIoO8g0NfnfL2MTJRkxoX
bfDaUeVPQuYEhg37NZWAJQ9VnMVDxP/VHL496M/QZxkjf5/Efucp2gaD
X6RS6CXpoY68LsvPVjR0ZSwzz1apAzvN9dlzEheX7ICJBBtuA6G3LQpz
W5hOA2hzCTMjJPJ8LbqF6dsV6DoBQzgul0sGIcGOYl7OyQdXfZ57relS
Qageu+ipAdTTJ25AsRTAoub8ONGcLmqrAmRLKBP1dfwhYB4N7knNnulq
QxA+Uk1ihz0=
```


Note that you should not copy it without verifying its authenticity, with DNSSEC, as in any cryptographic system based on a chain of trust, the importance of root items that are trusted by definition is capital. Its official distribution point is [at IANA](https://data.iana.org/root-anchors/), and the file itself is signed by GPG.
The command to run is as follows, we seek here to validate the IP of www.eurid.eu:

```
$ dig +sigchase www.eurid.eu
;; RRset to chase:
www.eurid.eu. 544 IN CNAME eurid.eu.
[...]
;; WE HAVE MATERIAL, WE NOW DO VALIDATION
;; VERIFYING DS RRset for eu. with DNSKEY:55231: success
;; OK We found DNSKEY (or more) to validate the RRset
;; Ok, find a Trusted Key in the DNSKEY RRset: 19036
;; VERIFYING DNSKEY RRset for . with DNSKEY:19036: success

;; Ok this DNSKEY is a Trusted Key, DNSSEC validation is ok: SUCCESS
```


The last line indicates that the validation was successful, because the chain of trust could be raised successfully and everything is fine, up to the known public key of the root zone.

If you get the following message is that dig did not find the root key in /etc/trusted-key.key:

```
$ dig +sigchase www.eurid.eu
No trusted keys present
```




## Method 3: In console mode, without prior submission of the root key
If you can not submit the public key as above, you can trust a third DNS server to validate for you. Some recursive DNS servers validating DNSSEC are made ​​available to the public by various entities. Note, for example those of [DNS-OARC](https://www.dns-oarc.net/oarc/services/odvr), that we take up in the example below, where we seek to validate the  IP of www.eurid.eu:


```
$ dig +dnssec www.eurid.eu @149.20.64.21

; <<>> DiG 9.7.3 <<>> +dnssec www.eurid.eu @149.20.64.21
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26117
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 6, AUTHORITY: 7, ADDITIONAL: 16
[...]
```


Here is the flag presence "ad" indicating that the response you receive has been validated by the recursive resolver.

