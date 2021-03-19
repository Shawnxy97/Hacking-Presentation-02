
**BLUEKEEP (CVE-2019-0708)**

**A Critical and Wormable RDP Vulnerability**

BlueKeep is a Remote Code Execution (RCE) vulnerability in Microsoft’s Remote Desktop Protocol (RDP) server, that affects Windows machines from Windows 2000 to Windows 7 and Windows Server 2008 R2. BlueKeep was found and patched in May of 2019. This vulnerability is a _use-after-free_ that was present in the Windows kernel driver that handles RDP connections (termdd.sys).

Successful exploitation of this vulnerability will result in an attacker being able to execute arbitrary code with Administrative (kernel-level) privileges. Based on the severity and criticality of this flaw, Microsoft released a patch even for unsupported systems such as windows XP and Windows Server 2003.

In the NIST website, BlueKeep reached a base score of 9.8 (Critical) when scored using CVSS version 3 and 10 (High) with version 2.0. That’s how bad it is. See below.

![CVSS Version 3](https://raw.githubusercontent.com/CincChou/Hacking-Presentation-02/main/CVSS3.png)
_CVSS Version 3_

![CVSS Version 2](https://raw.githubusercontent.com/CincChou/Hacking-Presentation-02/main/CVSS2.png)
_CVSS Version 2.0_

**What makes the vulnerability so critical?**

 1. It affects the RDP services used by millions of machines worldwide. This can be especially dangerous in critical industries such as health care and industrial controls.
 2. It allows for remote code execution
 3. It can be weaponized to be wormable. That is, the code exploiting this vulnerability can self-propagate and spread very quickly just like in the WannaCry ransomware attack in May 2017.

**How RDP Works**

The RDP service allows multiple users to interact with windows sessions remotely. By default, the protocol uses TCP Port 3389. RDP enables a connection between a client and server defining the data communicated between them in virtual channels. Virtual channels are bidirectional data pipes which enable data transfer between the server and the client.

On windows XP and server 2003, there are 32 virtual channels by default. Due to this limitation, dynamic virtual channels were created which are contained within a dedicated static virtual channel. Static virtual channels are created at the start of a session and remain until session termination, unlike dynamic virtual channels which are created and torn down on demand.

Windows binds the static virtual channel names to numbers within the driver _termdd.sys_ and more precisely within the functions __IcaBindVirtualChannels_ and __IcaRebindVirtualChannels._

Channels are referenced in windows with different names and can be used for different purposes. For example, CLIPRDR for clipboard sharing and RDPSND for sound sharing. These channels can be used by the client using the RDP API or application programmer interface.

**The Vulnerability**

By default, RDP reserves channel 31 with the name _MS_T120_ but does not check the existence of two channels with the same names. This is ultimately where the vulnerability lies. An attacker can set up this name with a different channel number resulting in the current RDP session having the same MS_T120 channel in two different places within 32 channels available. For example, the malicious MS_T120 could be on channel 20 while the default MS_T120 will be on channel 31. When the attacker sends crafted data through the malicious channel, the driver _termdd.sys_ will attempt to close the channel and terminate the RDP session. With the malicious channel closed the reference pointer will be cleared for channel 20 but a dangling pointer will remain tied to channel 31 leading to a _use-after-free_ vulnerability. A use-after-free let's an attacker predict space in memory to write arbitrary code and execute malicious payloads, in this case with kernel level privileges.

As can been seen in figure 1, the RDP Connection Sequence connections are initiated and channels setup prior to Security Commencement. This enables CVE-2019-0708 to be wormable since it can self-propagate over the network once it discovers open port 3389. [1]
![RDP Sequence](https://raw.githubusercontent.com/CincChou/Hacking-Presentation-02/main/RDP_Sequence.png)
_RDP Protocol Sequence_

**Steps to mitigate the Vulnerability**
 1. Patch as soon as possible with the latest Microsoft update.
 2. Disable RDP on non-sensitive systems and from outside of the network.
 3. Monitor incoming RDP connections or any attempts to write a custom channel with the name MS_T120
 4. Implement Network Level Authentication (NLA) which authenticates the user **before** the initiation of the RDP connection. This allows the server to dedicate resources only to authenticated users. _In the case of a critical vulnerability in the RDP protocol, NLA can limit the exploitation of this vulnerability to authenticated users only [2]._


**References**

[1] “[RDP Stands for “Really DO Patch!” – Understanding the Wormable RDP Vulnerability CVE-2019-0708]
[https://www.mcafee.com/blogs/other-blogs/mcafee-labs/rdp-stands-for-really-do-patch-understanding-the-wormable-rdp-vulnerability-cve-2019-0708/](https://www.mcafee.com/blogs/other-blogs/mcafee-labs/rdp-stands-for-really-do-patch-understanding-the-wormable-rdp-vulnerability-cve-2019-0708/)” Eoin Carroll, Alexandre Mundo, Philippe Laulheret, Christiaan Beek and Steve Povolny. May 21, 2019.

[2] “[Explain Like I’m 5: Remote Desktop Protocol (RDP)]([https://www.cyberark.com/resources/threat-research-blog/explain-like-i-m-5-remote-desktop-protocol-rdp](https://www.cyberark.com/resources/threat-research-blog/explain-like-i-m-5-remote-desktop-protocol-rdp))” Shaked Reiner. July 4, 2020.

