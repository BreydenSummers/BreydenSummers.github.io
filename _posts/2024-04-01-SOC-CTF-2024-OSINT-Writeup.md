---
layout: post
title:  "SOC CTF 2024 OSINT Writeup"
summary: "My solutions to all the problems in the OSINT Section of the in-house CTF for USU"
author: BreydenSummers
date: '2024-04-02 22:07:15 +0000'
category: CTF
thumbnail: /assets/img/posts/SOC-CTF-2024/header.png
keywords: CTF, Utah State University, OSINT
permalink: /blog/SOC-CTF-2024-OSINT-Writeup/
---

## 404 pond not found
For this one, we need to perform a reverse image lookup on Google for this. Once we do that we can see that it leads to a Wikipedia page about duck ponds. From here we can see a location for the image, Queen Elizabeth Park.

## Binary Lake
For this one we can just perform a whois lookup on this ip address 165.227.251.183. After doing that we can see this section:
```
OrgName:        DigitalOcean, LLC
OrgId:          DO-13
Address:        101 Ave of the Americas
Address:        FL2
City:           New York
StateProv:      NY
PostalCode:     10013
Country:        US
RegDate:        2012-05-14
Updated:        2023-10-23
Ref:            https://rdap.arin.net/registry/entity/DO-13
```
Seeing this we can use the orgname as our solution: DigitalOcean.

## The wiggle duckling 1 and 2
For this challenge, you need to go to wigle.net and submit the value AC:DB:48:48:84:47 as the BSSID. ![image](assets/img/posts/SOC-CTF-2024/2b016c99-2a88-4d4a-bd9b-2717ba9e1f3b.png) From here we can see that the network is found in Boston Public Garden. We can submit that as our solution for the first one. For the second one, we need to just google statues in the garden and we can see there is one in there called Make Way for Ducklings. We can submit that for the second challenge.


## ATT&CK!!! 1 and 2
For this one we can 
