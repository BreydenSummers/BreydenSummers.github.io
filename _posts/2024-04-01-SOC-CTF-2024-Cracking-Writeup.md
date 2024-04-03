---
layout: post
title:  "SOC CTF 2024 Cracking Writeup"
summary: "My solutions to all the problems in the Cracking Section of the in-house CTF for USU"
author: BreydenSummers
date: '2024-04-02 22:07:15 +0000'
category: CTF
thumbnail: /assets/img/posts/SOC-CTF-2024/header.png
keywords: CTF, Utah State University, OSINT
permalink: /blog/SOC-CTF-2024-Cracking-Writeup/
---

## a bird's worst enemy
For this challenge, we are given a sam file and a system file. Normally you could use a tool like sam2dump but since Windows updated those tools have been broken. One tool that works is mimikatz. Here is a good [article](https://sanjumalhotra26.medium.com/dumping-credentials-from-sam-file-using-mimikatz-and-cracking-with-john-the-ripper-and-hashcat-ce5bbf2f4f5a) on how to use mimikatz to dump the credentials. Once you are done with that you can crack the hash the same as shown in that guide. 
