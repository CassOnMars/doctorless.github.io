---
layout: page
title: "Building a Rust Phone, Part 1: Picking the Platform"
permalink: /2017-01-11-rust-phone-part-1
---

In [Part 0](/2017-01-08-rust-phone-part-0), I cover the justifications behind building a custom cellphone. In short, security cannot be guaranteed in the current offerings. I picked Rust as a language due to it being able to still work well at a low level, but has a more concise (read: fewer bugs) syntax than C and has better memory safety than C and C++. Before I jump into my (mis)adventures in _actually writing code_, there’s still one more aspect to consider — the hardware.

As mentioned in the prior article, I intend to base the phone on the Rust OS project, [Redox](https://www.redox-os.org/). If you’ve already participated in the project, or at least tried running it on your own hardware, you’d know it is still in its early days, and cannot run with full hardware support on much of anything. Knowing this, I’d have to still implement my own drivers for whatever hardware I choose, and potentially have to write my own bootloader, and any additional supporting assembly code if, say, an ARM processor was chosen. There are a lot of development ARM boards out there, most notably the Raspberry Pi. I think it’s great for certain purposes, but it’s computational power leaves something to be desired when compared to higher end phones. I’m not just wanting to build a custom cellphone for hobbyist use — I’m wanting it to be a reasonable replacement for my day-to-day phone — an iPhone 6+.

As part of the research I conducted in Part 0, I realized very quickly SoC solutions that merge the AP and BP are incredibly dangerous, and are entirely unauditable, and while some phones (like mine) _do_ physically separate the two components, literally the only guarantee that can be made by manufacturers to the inaccessibility of hardware (either directly or indirectly) to the BP is through _code_, which coincidentally, still can’t be fully audited (at best), or is completely unavailable (at worst). This is because, there does not exist a modern cell phone that has a physical (important distinction; code doesn’t count) killswitch to the baseband processor. Worse still, more and more manufacturers are making it impossible to remove (or replace) the battery, and so on top of the inherent danger of the BP (especially in SoC implementations), there’s the additional risk of power potentially being available to the BP and other hardware, even if the phone is “off”.

## Hardware Requirements

So for the hardware, I’ve got five major goals:

1. Find a way to adequately *separate the BP from the rest of the hardware* (preferably with a killswitch).
2. *All hardware must be supported with open source drivers* on Linux, so that there exists a reference for implementation on Redox (with a caveat, in that these drivers either must be separate from the Linux codebase with a compatible license for Redox, or I must keep the codebase of the driver implementation separate from Redox to avoid my contributions to Redox being responsible for poisoning the license with GPL’d code (if unfamiliar with the GPL, either look at the [FAQ](https://www.gnu.org/licenses/gpl-faq.html) or the [GPL (v2, for Linux)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html) itself. While some would argue that the vast difference in implementations should warrant no concern for license poisoning, there have been enough court cases over licensing issues explicitly involving the GPL that I would rather keep it separate if any concern _could_ exist, and wait for someone more qualified (a lawyer specialized in software licenses) to comment.
3. *I don’t want to slave my life away by making the challenge more difficult.* Since I’m intending on basing the software on Redox, I’d really rather not want to put more effort into simply porting it to a different architecture than what it currently supports.
4. *The hardware must be competitive for performance with my current phone.* And if I want to do that, I’ve got to match (or beat!) its specs:
	*CPU+GPU*: Apple A8 — 1.4GHz Dual Core ARM (ARMv8A instruction compatible, 64KB[data]+64KB[instruction] L1 cache[per core], 1MB L2 cache[shared], and a 4MB L3 cache [SoC-wide]), plus a 4 core PowerVR GX6450 (according to AnandTech)
	*RAM*: 1 GB LPDDR3
	*Baseband Processor*: Qualcomm MDM9625M (note that while this processor is physically separate from the CPU, it is virtually unable to be guaranteed [by the public] that it cannot obtain access to other hardware components besides the portion of memory it is supposed to)
	*WiFi*: 2.4GHz + 5GHz 802.11a/b/g/n/ac
	*Bluetooth*: 4.2
	*Storage*: 128GB
	*USB*: 2.0
	*Screen*: 5.5" IPS (1920x1080) Capacitive Touch Screen
	*Battery*: 2915 mAh
	*Camera*: 8MP back, 1.2MP front
5. *The complete device must be reasonably portable as a cellphone.* This is arguably the toughest problem out of the lot. I’m not saying I expect it’s going to be as slim as my current phone. Even lots of hobbyist development boards are excluded by this requirement either due to their shape or power needs. It’s not a replacement if I have to be stuck to an AC plug, carry around a 12 V car battery, or look like this guy:

![this guy](https://cdn-images-1.medium.com/max/1600/1*WPLRx0DVQrBIKC4PxgfWRA.png "this guy")
I’m a dedicated, paranoid hobbyist, but I still don’t want to be *that guy* on the metro.

I began the search by looking for a BP that was either easily removable (in a non-destructive manner), or could easily have a physical on/off switch put in place. In other words, it wasn’t going to be soldiered onto a board. While some offerings were available with a GPIO connector, I found this odd due to the bandwidth of data transfer over GPIO relative to the speed of LTE. I2C LTE modules exist, but it is far too low level, abstraction-wise, to be safe. The only other kind of bus that had available modules which also had sufficient bandwidth was USB, and as noted in Part 0, there are concerns with DMA attacks with that option. I left out a good deal of explanation behind how DMA attacks work, and how to prevent them in the prior article, because for one, adequately explaining it would be an article or two in of itself, and two, I had hoped that there may be a reasonable alternative to a USB device, but in reality, it seems to very expediently solve my need for a higher-level abstraction with a very obvious killswitch (unplug it). So without writing an article within an article, the TL;DR to preventing DMA attacks from USB on a hardware level is to ensure your processor has a form of IOMMU which supports interrupt remapping (let alone have an IOMMU at all).

## Narrowing It Down

A number of ARM development boards definitely cropped up, but most of them lacked in processing power to match my current environment, or was intended for other purposes to the point where the formfactor became unreasonable. I was rather excited after finding the MediaTek X20 development board, until digging into the deep, poorly-translated documentation revealed a dormant BP inside the SoC, along with other concerning elements. A similar experience was found with the very limited number of qualifying ARM boards.

x86? The thought of a strapping a laptop equivalent to my head to answer a call didn’t sound enticing.

![me guy](https://cdn-images-1.medium.com/max/1600/1*rU1G6bJ_JydSZlcEzhpgPQ.png "me guy")
This is unreasonable.

I kept searching for options, with some very exotic (read: difficult to support) processors, but kept coming to the same conclusion: x86 isn’t so bad. In fact, there are some small form factors, such as Intel’s NUC. And the Atom is pretty power efficient. But other than platforms that were too limited, like Edison and Galileo, options seemed out of reach — the Compute Card announced at CES isn’t available yet, and the docking station’s internals aren’t public yet, so waiting may not even be worth it. But then I remembered this sleeper car of a device: the Intel Compute Stick!

Here’s the specs on the lowest-end model of the latest (Q1 2016, sadly, but it is latest) iteration, the STK1AW32SC:

*CPU*: Intel Atom x5-Z8330 Processor — 1.92GHz quad-core, 24KB(data)+32KB(instruction) L1 Cache (per core), 2MB L2 Cache (shared)
*GPU*: Intel HD Graphics 400
*RAM*: 2GB DDR3L-1600
*WiFi*: 2.4GHz + 5GHz 802.11a/b/g/n/ac
*Bluetooth*: 4.2
*Storage*: onboard 32GB, microSD slot allows up to an additional 128GB
*USB*: 1 USB 3.0, 1 USB 2.0
*Video Out*: HDMI 1.4b
*Power In*: 5V, 3A max current
*Price*: $129

Perfect.

## Finding What Remains

This leaves us with needing to fill in the gaps: a touchscreen, a battery, and either one or two cameras (sticking with one for now), and of course, the baseband processor (plus necessary RF components).

While the world of touchscreens is somewhat of a mixed bag, many of the screens in cellphones take in some form of MIPI DSI or eDP connector, and even more fortuitous for someone wanting to start a project like this now, Toshiba has brought an HDMI to DSI chip (TC358870XBG) to market. Due to this, there is now a flood of excellent 1080P and 1440P screens in handheld sizes that I can use with this project, ranging from $100–200. At this point, however, I’m nowhere near ready to start implementing input for a digitizer or a camera, so I’ll leave that for a later article.

As for the battery, there exist many “power banks” on the market which can provide a 5V/3A output, some rated upwards of 30,000 mAh, all within very portable sizes. Again, there is little point in obtaining this yet, as I’m nowhere near ready to start taking this outside. Similarly, many USB BPs exist on the market, some of which have Linux drivers. This too, shall wait for another day.

Next up is Part 2: Attempting to Boot.
