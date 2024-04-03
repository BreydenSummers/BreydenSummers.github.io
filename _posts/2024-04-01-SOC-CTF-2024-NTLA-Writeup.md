---
layout: post
title:  "SOC CTF 2024 Network Traffic / Log Analysis Writeup"
summary: "My solutions to all the problems in the Network Traffic / Log Analysis Section of the in-house CTF for USU"
author: BreydenSummers
date: '2024-04-02 22:07:15 +0000'
category: CTF
thumbnail: /assets/img/posts/SOC-CTF-2024/header.png
keywords: CTF, Utah State University, Network Traffic / Log Analysis
permalink: /blog/SOC-CTF-2024-NTLA-Writeup/
---
## Brutalism 1,2, and 3
This is the [log](https://github.com/BreydenSummers/BreydenSummers.github.io/files/14847054/output.log)
 file for the challenge.
For the first one we can see that it is making a lot of web requests. From here we can just compare this to a bunch of web servers to find the right one. We'll find that the log type matches an NGINX server. For number 2 its pretty easy to see because it is the device that is flooding the logs with requests. You should find that the IP address is 181.225.59.149. For the last one you can use a command like this:
```bash
cat output.log | grep 181.225.59.149
```
This tells us how long the device was talking to the server. We can just take the time stamp of the last communication as the end of the attack and the first as the start of the attack. After putting it in the correct format you should get this: 13:47-14:02.


## Injection 1,2, and 3
For this problem we kind of need to solve all three challenges at once. From the log file, we can see that this is another web server. One of the questions is asking for the path of a file. From the name of the challenge we can think this is an injection attack. So with both of those bits of information, we can assume that the path of the file would be in the URL. Since URLs can't handle forward slashes being put into them willy-nilly we have to encode them. You can find a good reference for that [here](https://www.w3schools.com/tags/ref_urlencode.ASP). We can see that for the forward slash the code is %2F. Using that we can search for it and find the injection log. From there you can grab the ip address for 1, and grab the path for the attack for 2. Then you can also look at the log and determine that they were using UDP. 

## duck duck goose!
To start we can open up the pcap in wire shark to see what is going on. 
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/88056ad1-c644-4763-b21e-e9e75442f39b)<br>
From this, we can see a bunch of DNS requests, and prefaced for each request is the text dnscat. After looking that up we can see this [github repository](https://github.com/iagox86/dnscat2). From here we need to do some reading up on how this application works. After doing that we can see that some data is being transfered via this tool. From here we can use tshark (a tool for processing network captures from the command line) to get the data we want from the file and further process it until we get just the hex data out (just a note you can do this with Python as well I just opted to do it via the command line):
```bash
#Grab the relevant packets aka all the send queries to the c2 server
tshark -Y "dns and ip.dst == 192.168.1.146" -T fields -e "dns.qry.name" -r test2.pcapng >queries_out.txt


#grab everything but the first delimiter aka the dnscat
awk -F'.' '{for(i=2; i<=NF; i++) printf "%s%s", $i, (i==NF ? "\n" : "")}' queries_out.txt > mid.txt

#Remove the bits for networking information
awk '{print substr($0, 19)}' mid.txt > mid2.txt

# Remove all new lines
sed ':a;N;$!ba;s/\n//g' mid2.txt > mid3.txt

```
From here we can use cyberchef to see what is going on with the data. We can upload it to Cyber Chef and use the from hex option to look at the data. It should look like this: <br>
![Untitled](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/6b5ec882-15c4-428a-a165-5e623eab98a5) <br>
From here we can recognize the file marker for a png file. But it  looks like there is some garbage data out front. We can remove that until we get something that looks like this: <br>
![Untitled (1)](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/149909bc-bfa5-4878-88c2-5d3d16aec296) <br> 
Lastly, we can use the render image recipe from Cyber Chef to get our image and get our flag.
![Untitled (2)](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/118736f8-81d4-4533-8331-e90ecfb212bf)

