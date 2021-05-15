---
layout: post
title: The expected (and the correct) way to use SSD cache on Synology
date: '2019-12-16 02:52:05'
permalink: "/advices-of-ssd-cache-on-synology/"
tags:
- synology
- nas
- ssd-cache
---

I bought Synology 918+ and two Corsair MP500 120GB NVMe SSD in 2017. For just home usage, I just use NAS as daily backup and bit-torrent upload. Very light usage.

Recently, DSM said the SSD cache crashed. It happened a few times before. Each time it's on the first day of some month. DSM will do a data scrubbing on that day by default. During that process, the system will raise an error about SSD cache. I thought it was a compatiblility issue because as soon as I reboot the system (sometimes it had to be forced) and repair the SSD cache, it went back to work flawlessly.

> FYI, MP500 is not on the compatible list.

This time, it's different. I re-inserted that two SSD into M.2 slots and reboot, then start repairing. A few minutes after repairing finished, DSM said it got multiple I/O error on SSD cache. Then, every apps on Synology became unresponsive. I had to pull out the SSD and took them to a local dealer to find out whether it's a compatible issue or hardware failure. I ran Corsair SSD Toolbox under Windows and it turns out each SSD already have 10TB read and 190TB write. After discussing this situation with someone on [Reddit](https://www.reddit.com/r/synology/comments/e8m7yr/buggy_ssd_wr_cache_in_918_corsair_mp500_120gb/), it turns out that problem **does not lies in the SSD cache implementation** but lies in **the way I uses those SSDs**. The system doesn't actually write 190TB to the SSD but the [write amplification](https://en.wikipedia.org/wiki/Write_amplification) does.

**So here's my advice.**

- Try upgrading the RAM first before you consider using a SSD cache to speed up the whole system. 
- Always consider using a compatible SSD from the [official list](https://www.synology.com/en-us/compatibility). Enterprise-level SSD recommended, SLC is the best but it's the most expensive. 
- Read-only cache preferred. Since it will not increase any data loss risks and significantly more bang for buck. 
- In order to eliminated write amplification effect, Do not allocate entire size of the SSD for cache volume, leave at least 20% unused.

And that's what you should be aware of:

- SSD cache will not take participant in the sequential writes and reads. Most of activities are sequential workloads. 
- You should determine the bottleneck in your scenario before considering a SSD cache. The purpose of SSD cache is to offload some of the random access activities, so SSD cache should be effective if you're a heavy VM user.
- Pay attention to SMART data periodically. Life span ratio not accurate all the time. 

Some advices credited to [ssps](https://www.reddit.com/user/ssps/).

