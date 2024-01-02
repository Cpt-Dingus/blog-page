[//]: # (NOTE: I have explicitly requested permission from lego11 to use his guides as a refernece, he said he's cool with it. I owe that man half the shit I know)

# Preamble

Before anything, I have to give credit and extreme kudos to the SDR++ Discord. Everyone there has been incredible and has helped me learn basically everything that you'll be able to read here. 
**A lot of the information in guides is from [lego11s articles](https://a-centauri.com/articoli/), check their articles out if you want more awesome and detailed guides.** My goal is to make everything a bit more concise while still staying informative with a sprinkle of what I learned with experience as a complete beginner on top.

The guides here were written specifically to be up to date, since there are several WAY out of date guides out there instructing you to use abandonware and just overall shitty software.

This article will contain everything I know about receiving weather satellites.

# Glossary
Hardware terms
- SDR - Software Defined Radio -> A device used to translate radio waves into digital info
- LNA - Low Noise Amplifier -> A tool used to amplify radio signals
- Bias-t/Bias tee -> A device used to inject DC power into the rf line. **DO NOT PLUG IT IN AIMING AT YOUR SDR, IT WILL KILL IT**
- Sma -> Type of connector used by most SDRs
- Balun -> Converts an **Un**balanced signal to a **Bal**anced one and vice versa

Software terms
- AGC - Automatic Gain Control -> Automatically sets the gain based on the signal strength
- SNR - Signal to Noise Ratio -> The difference in dB between the noise floor and the signal peak, ergo how strong the signal is
- Spectrum -> The slice of the radio spectrum being sampled by your computer
- FSR or Waterfall -> A visual representation of the spectrum throughout time, most of the time found right below the Spectrum
- Interference -> Commonly referred to as RFI (Radio Frequency Interference), is unwanted signals produced by erroneous sources
- Overloading -> Occurs when your gain is set too high andor you are near a very strong broadcast. Presents as your noise floor jumping/being unstable or random spurs of interference throughout your spectrum


- TLE - Two Line Element set -> A format used to list the location of objects orbitting the earth
- Pass -> Refers to the time when you can see a satellite passing overhead, used with orbiting satellites
- Elevation -> Height of a satellite above the horizon

Don't worry if you don't understand the below terms, they will be elaborated upon in the article and are here just so you have an idea of what I mean if they are mentioned before they are explained.
- APT - Automatic Picture Transmission -> VHF Image broadcast format from NOAA satellites (As of writing this article)
- LRPT - Low Rate Picture Transmission -> VHF Image broadcast format from Meteor-M satellites (As of writing this article)
- LRIT - High Rate Information Transmission -> L-band information and telemetry broadcast format
- HRPT - High Rate Picture Transmission -> L-band high quality image and telemetry broadcast format


# Mistakes and pitfalls
This is a list of mistakes I made that made me waste a lot of time, make sure to avoid making them.

- **Using old, outdated guides and software** -> What is covered here is very niche, akmost all guides you can find online are heavily outdated suggesting you use abandonware that gives you subpar results or straight up giving you wrong advice. Please make sure that any sources you use are up to date.
- **Doppler tracking APT, LRPT** -> These signals were designed to be thin enough to not need doppler tracking. **DON'T BOTHER DOING IT, IT IS NOT NEEDED.**
- **Maxxing the gain setting** -> Upping the gain only makes the signal louder up to a certain point, after which it start amplifying the noise floor much more than actual signals. Turn it up only until you see the signal isn't getting stronger and the noise floor is rising rapidly.
- **Turning on automatic gain control** -> AGC was designed for wide broadcasts such as DVB-T (Terrestrial TV) which are much wider than the comparstively thin signals described in this article. It **won't** recognize the signals and end up cranking the gain up way higher than needed which ends up killing the signal
- **Using an LNA with direct sampling** -> Direct sampling is just piping the whole DS range to the output at once, meaning that weaker signals will get drowned out by stronger ones incredibly easily. Using an LNA makes strong signals stronger, drowning weak ones out completely.
- **Not using actual connectors and cables** -> In my first few days with an SDR and without an actual sma cable, I went by the definition of an antenna - An antenna is just a piece of wire - and stuck a wire straight into the sma port. While yes, sticking a wire into the sma port will work, it is extremely dangerous since it can damage your sma port, electrocute you if you have a bias-t, or just end up not working correctly. Just get proper cables, man
- **Using a cheap LNA with higher frequencies (l-band and above)** -> Cheap LNAs do work, but very poorly and are very often not worth the money spent. Check out the HRPT section for more info.


# Preferred software

Arguably the by far best choice for **pulling data** off of satellites is:
- [Sdr++](https://github.com/AlexandreRouma/SDRPlusPlus/releases) for recording
- [SatDump](https://github.com/SatDump/SatDump/releases) for decoding, or just recording (covered in hrpt). 
Always download the nightly builds for both of these, since the tools are relatively new and are being actively developed with new additions basically daily. There are other programs you can use but I won't focus on them for the sake of keeping this simple.
---
 To **track satellites** and figure out their future passes (Most are orbiting the Earth after all), you can use these:

- [Orbitron](http://www.stoff.pl/) - Windows - Quite dated but functionally sound
- [Gpredict](https://oz9aec.dk/gpredict/) - Windows, Linux, MacOS - An much more recent tool, preferred in most cases
- [Look4Sat](https://play.google.com/store/apps/details?id=com.rtbishop.look4sat&hl=en&gl=US) - Android - Provides everything essential in a simple UI.
- For IOS there are a few apps but they have severe limitations, using any of the above is preferred
I personally use Look4Sat when on the go because of its simplicity and Orbitron when at home, since it is able to simulate a map of all satellites from any point in time and I just got used to its UI.

*Make sure you update your TLEs*, not doing so might make your data be outdated or just outright incorrect.



# VHF satellite reception guide (APT & LRPT)
- Receiving VHF broadcasts is incredibly easy -> all that is needed is just some wire, an SDR and some patience
- As of writing this article there are just **4** remaining weather satellites that broadcast in this band
- While easy to receive, they have a lower quality (4km/px and 1km/px with jpeg compression) and transmit less channels (2-3) than broadcasts in higher frequencies which usually transmit 10+ channels of raw 1km/px images

## The four weather satellites still broadcasting images on VHF

This is a brief historical overview in case you want to know a bit about the satellites you can receive. Feel free to skip this heading if you are just here to receive stuff.

---

- The **3 NOAA satellites** are the last remaining members of the **POES** (Polar Orbiting Environmental Satellites) constellation, consisting of **NOAA 15, 18 and 19** being launched in 1998, 2005 and 2009 respectively.
- New satellite launches are of the JPSS constellation, which only have a much harder to receive C band signal that requires a big 'ole dish with a helical antenna and specific hardware to receive (Since most SDRs only go up to 1.7 GHz). No future satellite launches from NOAA are planned to include a VHF antenna.

- These satellites broadcast an *analogue*  **[APT (Automatic picture transmission)](https://www.sigidwiki.com/wiki/Automatic_Picture_Transmission_(APT))** signal, which was created in the late 1970s. Its analogue nature means, that if the signal has even just a bit of noise, you will get static grain on output images.

---

- As for their Russian counterpart, the **only** satellite currently broadcasting in VHF is **Meteor-M N2-3** (Meteor M2-3 for short) of the **Meteor-M** constellation. It was launched very recently - just in June of 2023. 
- The satellite series has been plagued with errors, failures and delays. M2-3 is sadly no exception: its LRPT antenna didn't fully extend, leaving it in a tilted angle making the signal not circularily polarized like it is supposed to be as well as making it much weaker than designed. M2-4 is currently set to launch 02-2024.

- Meteor-M satellites broadcast a *digital* **[LRPT (Low rate picture transmission)](https://www.sigidwiki.com/wiki/Low_Rate_Picture_Transmission_(LRPT))** signal, which includes **ECC** to make sure the picture doesn't come out grainy as well as allowing you to decode the signal properly even if it is weak.


## Exemplary images
![An exemplary processed APT image](https://github.com/Cpt-Dingus/Obsidian-notes/assets/100243410/a0a286a7-964f-4fc1-b558-5ce9f4233876)
*APT image received using a 5 element yagi on 01-01-2024 from NOAA 19, processed using Satdumps `MCIR RAIN` RGB composite. Equalized.*



![An exemplary processed LRPT image](https://github.com/Cpt-Dingus/Obsidian-notes/assets/100243410/3b20bba0-ae3d-4211-8a97-9c4abe58e8d8)
*LRPT image received using a 5 element yagi on 31-12-2023 from Meteor M2-3, processed using Satdumps `224` RGB composite. Equalized. The length is not to scale with NOAAs broadcast, this was just a poor pass.*



## Hardware needed to receive these satellites

You will need an SDR and an antenna, no other special equipment is required for VHF.
The SDR should be a **reputable brand** if you want optimal performance (E.g. AirSpy, RTLSDR Blog, Nooelec...). In simpler terms; avoid aliexpress chinesium sdrs.

As for the antenna, you have the choice between:
- Directional antennas
- Omnidirectional antennas
The difference is, that with directional antennas you need to track the satellite by hand, while an omnidirectional antenna doesn't need any tracking.


Popular antenna types for receiving in this frequency include:
### 1. V dipole antenna
- Omnidirectional
- Very easy to make, very portable
- Fair results, has some nulls due to its inconsistent polarization
- Arguably the best for beginners
- When using, point directly north/south & move about 50 cm from the ground (See when the signal is the strongest, play around with it and stick with what works!)
  
To build it: 
1. Get a chock block (electrical terminal), a coaxial cable and two preferably copper, unshielded wires that are ~54.5 cm long (This is because the v dipole elements have to be a fourth the wavelength, which in this case is `299,792,458 ms⁻¹/137,500,000 hz = ~2.18 m; 2,18/4 = ~0.545 m = ~54.5 cm`. Why? [It's how a v dipole works.](https://upload.wikimedia.org/wikipedia/commons/d/d8/Dipole_antenna_standing_waves_animation_6_-_10fps.gif))
2. Stick the shielding of the coaxial cable in one hole and the copper core into the other (keep it as short as possible)
3. Put the two wires into the holes and spread them 120° apart making a V shape.
That's it. Really.

![A visual guide on how the antenna should look](https://github.com/Cpt-Dingus/Obsidian-notes/assets/100243410/3d404bc7-4f77-4541-bda6-c851c16109fb)

*This image suggests a wire length of 53.4 cm, while that would work the actual length should be approximately 54.5cm. It also suggests using aluminum rods, while that'd work, copper is about twice as conductive.*

#### 2. Quadrifilar helical antenna
- Omnidirectional
- Fairly difficult to build
- Very good results thanks to its true circular polarization
- Best choice for permanent fixtures
  
I will not be describing how to build it, since the building process is quite involved. There are plenty of guides out there, if I ever get around to building one I will link it here.

![Sample QFH antenna](https://github.com/Cpt-Dingus/Obsidian-notes/assets/100243410/0f50f247-228b-46b7-961b-bb0bb2327a12)
*[source](https://okelectronic.wordpress.com/2014/08/20/rtl-sdr-second-attempt/)*

#### 3. Yagi antenna
- Directional
- Fairly easy to make
- Very good results thanks to its high gain
- Requires manual tracking
  
![An image describing the parts of a yagi antenna](https://github.com/Cpt-Dingus/Obsidian-notes/assets/100243410/f180d5ac-ee7e-461d-8aa3-84ca5ff0411a)
*[Source](https://duo.com/labs/tech-notes/the-yagi-uda-antenna-an-illustrated-primer)*

I personally use this type of antenna for VHF passes, since it constantly gets a great SNR (~45-50) and is quite easy to build albeit requiring a bit more wire. It also helps with coping with only being able to receive one satellite at a time, since L-band needs a dish and can only get one satellite at a time as well.

To make it:
1. Decide how long you want the antenna to be
More elements = More directional but higher gain, longer boom length
2. Shove appropriate numbers into [this calculator](https://www.steeman.org/Antenna/Yagi-Antenna-Calculator) (For frequency choose 137.5 MHz)
3. Cut some copper wires, place them according to to the values from the calculator
4. For the driven element (dipole), cut it in half and put one side in a terminal with the shielding of a coaxial wire and the other with the copper core of the same coaxial wire\
**Make sure the cables don't touch or are shorted together in any way, this will make the antenna not work**
5. Coil the coaxial cable up a few times right after the feed point in order to convert the **unbalanced** signal to a **balanced** one (The goal is to create a choked balun)
![An image of the choked balun I use on my yagi](https://github.com/Cpt-Dingus/Obsidian-notes/assets/100243410/785761ee-3536-44b2-af42-64c03a3e73f6)
*The choked balun I got on my yagi, I will update this with a better picture when possible.*


## Frequency reference

As of 26.12.2023, the frequencies these satellites broadcast in are as follows:

|Satellite|Frequency|
|---|---|
|NOAA 15|137.62 MHz|
|NOAA 18|137.9125 MHz|
|NOAA 19|137.1 MHz|
|Meteor M2-3|137.9 MHz|

## Actually receiving the satellites!
1. Get to a place with a good view of the sky - The more you can see, the longer you can receive the satellite for and the longer the resulting image will be
2. Configure SDR++ as follows:

- Rise gain until your noise floor starts to rise more than the target signal, or until before your SDR overloads, whichever comes first. Try not to move it around more than once. In my experience it is stable at around 30 dB gain.

#### FOR NOAA APT
- Move your radio to the correct frequency (refer to the frequency chart provided above)
- Select NFM with a bandwidth of about 45000 Hz
- Tick `IF noise reduction` and slect `NOAA APT`
- Turn the low pass filter on, disable the high pass one if it was enabled
- Choose `Audio` as the recording type and once the sat goes into view hit `Record`

#### FOR METEOR-M LRPT
- Go to module manager, add a `Meteor demodulator`
- Turn it on, tick `OQPSK`
- Move it to the correct frequency (As of writing this article, just 137.9 MHz)
- Hit `Start` n the demodulator tab

3. Move your antenna into place\
Tip if you are using a yagi: Go by signal strength, NOT by elevation and azimuth. Use apps to find the time and general direction of the satellite, don't go measuring exact compass readings.
Move slowly and try keeping the signal as loud as possible. You might not get the hang of it on your first try, tracking is a skill you have to learn!

If everything is right, you are now receiving a beeping APT signal or you see four dots on the demodulator if it is LRPT!

>The signals are usable when the SNR is over:
> - ~20 dB for LRPT
> - ~25 dB for APT

4. Once the pass is over, hit `stop` on the recorder or demodulator

## Converting the recording to an image
1. Open SatDump, in the `Offline processing` tab select:
- `Meteor-M2-x 72k LRPT` and `soft` as the file type, put the .s file in the input (NOTE: The symbol rate [72k] might be incorrect depending on the current satellite status, as of 01-01-2024 the correct rate is 72k)
- `NOAA APT` and `audio_wav` as the file type, put the recording in the input
  
You don't have to set the timestamp manually, satdump will figure it out from the recording file.

2. Hit `Start`, satdump will now demodulate andor decode the input file
3. Once it finishes processing, head to the `Viewer` tab and select the pass you decoded on the top left.

You are done! Feel free to play around with the image settings and enhancements, you can figure it out :)

## Common issues
- The image is solid black!
If the image was taken from Meteor M2-3 on an evening pass, you need to select Channel 4 (IR) to see anything - Channels 1 and 2 are visible, during night it is solid black

- There is grain all over the image!
Some grain is expected on APT images, you can get rid of it by ticking `Median blur`. If it is present after, either you didn't enable the IF noise reduction when recording or the signal was just too weak.

## L-band HRPT/HRIT reception

TODO
