---
layout: default
title: HamGeek SDR starter guide
nav_order: 2
parent: Radio
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Preamble

Hamgeek has recently begun selling cheap SDRs capable of very respectable sampling rates, however they require some tinkering to get started with. This guide will list out the different types available, how to get them to perform the best they can. LibreSDRs are included since they are the original design, however the primary focus of the guide is the cheaper HamGeeks.

> Huge thanks to @mothmaux for helping with the different variants as well as overclocks, this article wouldn't exist without him.

# Variants

These SDRs all come with an AD9363 or AD9361 tuner, which are rated at up to 61.44 Msps with a range of 70 MHz - 6 GHz
 
> Note: AD9363 is officially rated as 325 MHz - 3.8 GHz, but still work at the same range as the AD9361 albeit at a reduced sensitivity.

There are two major types of the SDRs: 
- The original PCB design - **LibreSDR***
- The newer and cheaper design - **Fishball** made by **HamGeek**

LibreSDRs consistently outperform the HamGeeks in sampling rates, the latter of which is able to overclock significantly higher which is required to compete.

However, by far the biggest factor that sets these apart from one another is the CPU chipset, which can limit the maximum stable sampling rate. The different SDRs are:

## HamGeek (Fishball) - Zynq70x0

- These can be purchased from both their official [website](https://www.hgeek.com/) as well as [AliExpress](https://www.aliexpress.com/store/1102732208). They appear to be clones of the LibreSDR devices, with some cut corners as well as cheaper components to lower the price. Most notably they use the AD9363 tuner which perform a bit worse than the AD9361 of LibreSDRs.

![An image of a Zynq7020 HamGeek.](https://www.hgeek.com/cdn/shop/files/92589-1.jpg)<br>
*Zynq7020-based HamGeek*
 
### Zynq 7010

- €99 [Official store](https://www.hgeek.com/products/hamgeek-70mhz-6ghz-zynq7010-ad9363-sdr-software-defined-radio-development-board-for-pluto-sdr-matlab?_pos=3&_sid=4fa857b41&_ss=r)
- €108 [Official store](https://www.hgeek.com/products/hamgeek-70mhz-6ghz-zynq7010-ad9363-sdr-software-defined-radio-development-board-with-shell-for-pluto-sdr-matlab?_pos=2&_sid=ac829a74d&_ss=r) - With case
- €113 [AliExpress](https://www.aliexpress.com/item/1005008303508008.html)

This is the cheapest variant of these SDRs, being available for as little as €99 from the official store. Most people have achieved around 35 Msps stable, though the maximum sampling rates appear to be based on silicon lotteries. Please note that the version with the case is known to have issues with oscillations, purchase some RF-absorbing foam and put it inside if possible.

### Zynq 7020

- €115 [Official store](https://www.hgeek.com/collections/software-defined-radio/products/hamgeek-70mhz-6ghz-zynq7020-ad9363-sdr-software-defined-radio-development-board-for-pluto-sdr-matlab-1)
- €150 [AliExpress](https://www.aliexpress.com/item/1005008162884638.html?)

This variant is able to pull around 45 Msps without drops for most people, is able to do it more reliably than the Zynq7010-based variant. Being priced just above the Zynq7010 and being sold with included cooling, it is arguably the recommended option.

## LibreSDR - B2x0 / Zynq7020

- These can be bought from the [OpenSourceSDRLab](https://opensourcesdrlab.aliexpress.com/store/4586015) website as well as their [AliExpress](https://opensourcesdrlab.aliexpress.com/store/4586015) store.

![An image of a B210-based LibreSDR](../../assets/images/hgeek/libresdr.jpg) <br>
*B210-based LibreSDR without a case*

### Libre (Zynq) 7020

- €135 [Official site](https://opensourcesdrlab.com/products/libresdr-zynqsdr-ad9363-zynq7020)

Functionally identical to the HamGeek Zynq7020, just performs better due to a better PCB design as well as the slightly more sensitive AD9361 tuner. An AD9363 version is also available but there's really no good reason to get it.

### Libre B210

- €207 [Official site](https://opensourcesdrlab.com/products/libresdr-b210-mini-ad9361)
- €260 [AliExpress](https://www.aliexpress.com/item/1005009175889484.html)

This SDR variant comes with improved connectivity - USB 3.0 as well as 5 gigabit LAN. This allows it to reliably pull the full 61.44 Msps even without overclocks.

### Libre B220

- €233 [Official site](https://opensourcesdrlab.com/products/libresdr-b220-mini-ad9361)
- €270 [AliExpress](https://www.aliexpress.com/item/1005007542400624.html?)

Effectively the same as the B210. It is theorized that a firmware tweak can be made to allow 122.88 Msps much like on the BladeRF SDRs, but this hasn't been attempted yet.



# Accessing the SDRs

Since these SDRs are essentially small computers, they have a fully fledged Linux shell inside that you can SSH into. 
> The user account is `root` and the password is `analog`.

To access this shell you can use:

## Standard Ethernet (Zynq70x0)

If plugged into a DHCP-capable device, the SDR assigns itself an IP that you can use as `ssh root@<ip>` like any other computer.

## Ethernet over USB

When plugging the SDR in using the standard USB port, you should see a new network connection pop up with the IP `192.168.2.1`, said IP is the IP of the SDR. You can log into it using `ssh root@192.168.2.1`.

## Debug USB

If all else fails or you messed some configs up, you can always use the JTAG USB port marked as `Debug`. You can plug into it and use the TTY to log into the shell following onscreen directions. To use the TTY you can use a program like `picocom` as such: `sudo picocom -b 115200 /dev/ttyUSB#` where `#` is the number of the SDR's TTY. 

> It's usually `/dev/TTYUSB1`.


# Getting the high sampling rates on Zynq 70x0-based SDRs

> The B2x0-based SDRs can do high sampling rates out of the box, this heading only applies to the Zynq SDRs! On B2x0, just use CS8 and you'll be fine.

When you first unbox the SDR and plug it in, it is likely to not be able to do much more than some 10 Msps. This is, because the stock firmware is quite limiting - the AD936x tuner can do much better. We can push it by doing the following:

## 1. Installing custom firmware

To access to some features, we can first install custom firmware, most notably [Tezuka](https://github.com/F5OEO/tezuka_fw/) which is what this guide will focus on. Most notably it includes

To install it:

#### 1. Head to the [Releases](https://github.com/F5OEO/tezuka_fw/releases/latest/) page and select your SDR accordingly

> Note that `fishball` refers to Zynq 7010-based SDRs and `fishball7020` refers to Zynq 7020-based SDRs

#### 2. Remove the SD card from the SDR

While some guides recommend you flash the SDR, it is a dangerous process that can lead to you bricking it. Using the SD card is 100% safe.

#### 3. Unzip the contents of the file you downloaded onto the SD card

Back the original ones up in case you ever need them.

#### 4. You are done!

Upon restart, the SDR should prompt you with the Tezuka logo on the shell and the website (accessing the IP or `fishball.local`) should show Tezuka firmware.

![Tezuka greeting on the shell](../../assets/images/hgeek/tezuka_greeting.jpg)

## 2. Use CS8 streaming

Tezuka exposes the AD936x chipset's ability to stream at a CS8 bit depth, this lowers the dynamic range of the SDR at the benefit of doubling the maximum sampling rate: it normally uses CS16, which streams 16 bits per sample whereas CS8 only steams 8. You won't be able to see a difference in the vast majority of use cases, 

Using this setting effectively doubles the sampling rate, as you only use half the data rate. 

## 3. Use the Ethernet port

The Zynq based SDRs only have a USB 2.0 chipset limiting the throughput at just 480 mbps, making the maximum theoretical sampling rate just 30 Msps on CS8! You should ALWAYS use the Ethernet connector with a gigabit connector, preferably directly.

When using directly, you must set the IP manually on both the SDR and your computer:

On the SDR:
- Run `ifconfig eth0 192.168.1.10`

On your computer: 
- Gateway: `192.168.1.1`
- IPv4: `192.168.1.99`
- Netmask: `255.255.255.0`

You can make the SDR-side config persistent by following [this article]({{site.baseurl}}/docs/Radio/Persistent HamGeek Ethernet IP).


> Warning: Even though apps might mark the SDR as using `IP`, you might be using the Ethernet over USB feature! I personally recommend you always just plug into the JTAG port, since that doesn't create said interface preventing possible confusion.

## 4. Overclocking

Tezuka allows us what every PC enthusiast salivates at the thought of: **overclocking**. It is done by modifying CPU and RAM multipliers, which, when multiplied by 25, give you the target clock speed in MHz. Future numbers will be said multipliers unless stated otherwise.

By default, these SDRs have their CPU run at just 750 MHz (30x) and the memory at 525 MHz (21x), the goal is to push it as high as possible where it still boots but does not kernel panic.

> Please note that you should absolutely use heatsinks with a fan when doing heavy overclocking!

Thanks to @mothmaux managing to reverse engineer how the overclocks are applied, he has created [this repo](https://github.com/ModderMax/O-C-Scripts-for-Tezuka_FW) which contains some premade files for all relevant Tezuka firmware variants, including a script to create your own.

To overclock the SDR:

#### 1. Remove the SD card from the SDR
#### 2. Replace the BOOT.bin file with an overclock file
#### 3. Put the SD card into the SDR, start it up
#### 4. Increment the sampling rate in SatDump until you begin seeing `PlutoSDR underflow!` errors. 

- These occur when you drop a sample. Pick the sampling rate closest to it that doesn't have these periodically appear.

> Make sure your computer is running in high performance mode and is under the lowest load! Use Linux if able.

#### 4. SSH into it and watch the terminal for kernel panics
#### 5. Depending on the result:
- If the SDR remains stable after a few minutes without underflow errors, you can bump the overclock up
- If the SDR underflows, use a lower sampling rate. If it's too low for your liking, increase the overclock or make sure your computer has enough resources for the chosen sampling rate. If possible, use Linux.
- If the SDR suffers a kernel panic and restarts, lower the overclocks


> Please note that the SDR might not boot at all with the blue `DONE` LED never lighting up, this happens when an overclock you selected is too large. Use a lower one until it's happy again.

Repeat this process until you find the 'wall' - place where you can't go higher with overclocks because of kernel panics. If the SDR runs for a few minutes without dropping samples, congrats! You have successfully overclocked the SDR.

# Cases

The LibreSDRs all come with cases, HamGeek offers one with the Zynq7010 version (Both machined aluminum as well as .STEP files for 3d printing), however the **Zynq7020-based HamGeek doesn't come with one**.

I have designed one, it can be found [here](TODO)

---

You should now be good to use your SDR with the optimal performance. Have fun!

