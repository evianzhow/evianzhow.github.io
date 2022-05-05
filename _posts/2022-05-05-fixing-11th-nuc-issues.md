---
layout: post
title: Fixing 11th NUCs occasionally not powering off completely issue (black screen, have to do a forced shutdown)
date: '2022-05-05 14:40:00'
permalink: "/fixing-11th-nuc-issues/"
tags:
- NUC
- Tiger Lake
---

For years, I’ve been a Mac user. macOS provides a perfect work environment for all programmers. But due to performance issues, I wanna give Wintel another try. last year I’ve bought a Tiger Lake NUC and expected the whole system to run smoothly under 'modern' 4K resolutions. My monitor was the mainstream DELL 2720Q, which connected to the NUC using an HDMI port. It's really a powerful, quiet and energy-efficient unit for daily uses. But I still encountered some weird issues. 

**In summary, the symptoms are (all happen occasionally):**

1. Screen blacks out, but not completely powered off after I clicked 'Power Off' in the startup menu, have to do a forced shutdown.
2. If the system went into idle and sleep, it will reboot unexpectedly. I checked the 'Windows Reliability Monitor' but found no detailed information.
3. Boot into the login screen, leaving the screen to be timed out. The monitor will not wake up from the keyboard or mouse.  

The above issues both occur on Windows 10 and Windows 11. At first, I thought it might be an operating system compatibility issue, something related to Intel Management Engine, or else I’d bought a faulty unit. After searching the web, I finally found something useful. From the [Chiphell](https://www.chiphell.com/thread-2406048-1-1.html) forum, many 11th NUC owners experiencing the same issues. All clues pointed to the latest driver for Xe dedicated graphics card. I tried downgrading the version, reverting to 27.20.100.9466, and now the system works fine (under ~30 hours of test).

So here’s how to I solved it (software method):

1. Rolling back Intel Xe Graphic Card drivers to 27.20.100.9466.
Download the version 27 drivers from [here](https://www.intel.com/content/www/us/en/download/19344/30381/intel-graphics-windows-dch-drivers.html).
2. Since the windows update might install a later version of graphic cards (30.0.100.9684 and newer), we should exclude driver receiving updates from Windows Update through 'Group Policy Editor'
```
*  Use the Windows key + R keyboard shortcut to open the Run command.
*  Type gpedit.msc and click OK to open the Local Group Policy Editor.
*  Browse the following path: Computer Configuration > Administrative Templates > Windows Components > Windows Update
*  On the right side, double-click the Do not include drivers with Windows Update policy.
*  Select the Enabled option on the top-left.
```
If you still want other components to receive the latest driver updates, you can try the [Intel® Driver & Support Assistant](https://www.intel.com/content/www/us/en/support/intel-driver-support-assistant.html) tool. 
3. (Optional) in case of accidental driver update, I recommend blocking driver installations for Xe dedicated graphic cards:
Steps on: [https://www.ghacks.net/2017/06/03/stop-windows-from-installing-drivers-for-specific-devices/](https://www.ghacks.net/2017/06/03/stop-windows-from-installing-drivers-for-specific-devices/). 
Be sure not to check `Also apply to matching devices that are already installed`, otherwise you have to downgrade again.

**The hardware method is pretty straight forward.** The only step is to use DisplayPort instead of HDMI, If you have to connect your monitor using HDMI, try an adapter instead. 
 
I hope this article would help you. 
