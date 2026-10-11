Disclaimer: I cannot document every step of the room per TryHackMe policies as they do not generally permit normal users to make such guides so in places where i cannot place a screen shot i will explain my thought pattern and reasoning at that point.
Summit room
Objective:

A company called Picosecure has decided to organize a threat detection simulation to bolster its malware detecting capabilities. We have been asked to work with a pen tester wherein the penetration tester will execute malware samples on the simulated internal user work station and we will configure Picosecure’s tools to detect the malware. We will use the pyramid of pain which we learned from the earlier pyramid of pain room as a guide here.
![](Pasted%20image%2020260928114413.png)

This is the lab machine TryHackMe has setup for us:
![](Pasted%20image%2020260928103407.png)Our first objective is to scan the file sample1.exe with Malware sandbox tool and go through the generated report:
![](Pasted%20image%2020260928103451.png)In the following report the file came back as suspicious and Malicious
![](Pasted%20image%2020260928111206.png)
Using the hash of the file which we got from the report we will look up this file on Virus Total to understand how the malware works and also we will block its Hash using the manage Hashes tool on the site
![](Pasted%20image%2020260928113048.png)For our purpose we will use the SHA-256 hash algorithm as collisions are significantly more unlikely compared to legacy algorithms making it more unlikely that a harmless file would be blocked instead.
We have successfully blocked sample1.exe however the penetration tester has said that due to weakness of hashes wherein he needs to only alter a single bit in the file to change the hash it will be very easy for him to bypass this rule and has asked us to find another way to block sample2.exe.
![](Pasted%20image%2020260928114244.png)
Going up the pyramid of pain the next step calls for us to target the attacker's Ip addresses so that is what we shall do. Now that we have the Ip address associated with sample2.exe we will block it using the firewall Manager.
![](Pasted%20image%2020260928120150.png)
We have successfully blocked the IP address behind sample2.exe however as the pen tester has pointed out it is quite trivial for an adversary to change his IP address so he simply signed up for a cloud service provider and now has  access to multiple more IP addresses. So now he has asked us to find a more concrete way to stop him so now moving up the pyramid of pain we will address the domain names used by sample3.exe.
![](Pasted%20image%2020260928122001.png)
The domain name being used by the attacker is clearly visible on the report so we will simply block it using the DNS rule manager in the VM
![](Pasted%20image%2020260928122159.png)The penetration tester has contacted me again and this time he says even though blocking an domain name is somewhat problematic for an adversary he can still just purchase a new one and has presented us with sample4.exe.
![](Pasted%20image%2020260928122844.png)
So in accordance with the pyramid of pain we will move towards the adversary's Network/Host artifacts.We can see in the Registry Activity part of the report that this malware does the following:
1/Disables rea time monitoring
2/EnableBalloonTips
3/progid
We will use sysmon logs option in the Sigma Rule Builder in the Vm to build a detection rule around this.Using the Registry Key,Registry Name, Value and Attack id.
![](Pasted%20image%2020260928124127.png)
Now we have successfully configured rules to stop the malware.Now according to the pen tester he says "I finally have `sample5.exe` for you to detect. Different approach this time. In this sample, all of the "heavy lifting" and instruction occurs on my back-end server, so I can easily change the types of protocols I use and the artifacts I leave on the host".So now in order to counter his new approach we will move up the pyramid of pain into the tools level. As a hint he has given us the logs for the outgoing network connections of the last 12 hours.
Using the patterns which are the amount of data being sent out as well as interval we use the network connections sub-category under the sigma rule tool to block the adversary.
Now onto the final sample sample6.exe where the penetration tester has asked us to act according to final level of the pyramid of pain and target his TTPs and for this purpose he has given a command logs document going over the commands he has used in all his previous attacks.Using the commands we can notice a file he uses in his attacks and we will build a sigma rule around the file modification sub-category to counter it using the file name and path from the logs.
![](Pasted%20image%2020260928151706.png)
