# Preamble

Before I write anything in here, I have to give extreme props to the SDR++ Discord, everyone there has been incredible and has helped me learn basically everything that'll be said in here. A lot of this information is from [lego11s articles](https://a-centauri.com/articoli/) (Extreme respect to him), I am just collecting everything I know in one place for future reference or if I ever revisit this hobby.

Parts of this can also just be considered a more concise & up to date guide to receiving sats, since most articles out there are faaar out of date (Looking at you, rtlsdr blog with 2007 instructions)


# Satellite reception

Below is everything I know that is specific to weather satellite reception, as well as a guide for them. I am yet to attempt anything else as of writing this.

## Pitfalls
This is a list of mistakes I have done that should be avoided

- Doppler tracking APT and LRPT -> These were designed to be thin enough to not need doppler tracking. **DON'T BOTHER DOING IT, IT IS NOT NEEDED.**
- Turning on automatic gain control with APT and LRPT -> AGC was designed for wide broadcasts, it will crank the gain way higher than it has to be raising the noise floor
- Turning the gain all the way up -> Only turn the gain up as long as the signal is getting louder and the noise floor isn't, don't just crank it all the way to the right
- Using an LNA with direct sampling -> Direct sampling is just piping the whole range to the output at once, meaning that weaker signals will get drowned out incredibly easily
- Not using actual cables -> This is by far the dumbest thing I have done. While yes, sticking a wire into the sma port will work, it is extremely dumb and will lead to a plethora of issues. Just get a proper SMA cable, man

## VHF reception guide (APT & LRPT)

- This was heavily inspired by [lego11s article on it](https://a-centauri.com/articoli/noaa-poes-satellites-reception), I tried to make this a bit more concise. 100% visit the article if you want more info than is provided here.
[//]: # (NOTE: I have explicitly requested permission for this from lego11, he said he's cool with it. I owe that man half the shit I know)
- Receiving VHF broadcasts is incredibly easy -> all that is needed is just a wire, some patience and an SDR 
- As of writing this article there are just **4** remaining weather satellites that broadcast in this range.
- A downside to VHF broadcasts is, that they have a lower quality (4km/px and 1km/px with jpeg compression) than broadcasts in higher frequencies (usually raw 1km/px)



### The few satellites still broadcasting VHF

This is a brief historical overview in case you want to know a bit about the satellites you can receive. Feel free to skip this heading if you just want to receive stuff.

---

- The **3 NOAA satellites** are the last remaining members of the **POES** constellation, consisting of **NOAA 15, 18 and 19** being launched in 1998, 2005 and 2009 respectively. 
- New satellite launches are of the JPSS constellation, which only have a much harder to receive X band signal that requires a big 'ole dish with a helical antenna to receive. No future satellite launches from NOAA are planned to include a VHF antenna.

- These satellites broadcast an *analogue*  **[APT (Automatic picture transmission)](https://www.sigidwiki.com/wiki/Automatic_Picture_Transmission_(APT))** signal, which was created in the late 1970s. Its analogue nature means, that if the signal has even just a bit of noise, you will get static grain on output images.

---

- As for their Russian counterpart, the **only** satellite currently broadcasting in VHF is **Meteor M2-3** of the **Meteor-M** constellation. It was launched very recently - just in June of 2023. 
- The satellite series has been plagued with errors, failures and delays. M2-3 is sadly no exception: its LRPT antenna didn't fully extend, leaving it in a tilted angle making the signal not circularily polarized like it is supposed to be as well as making it much weaker than it is supposed to be.

- Meteor-M satellites broadcast a *digital* **[LRPT (Low rate picture transmission)](https://www.sigidwiki.com/wiki/Low_Rate_Picture_Transmission_(LRPT))** signal, which includes **ECC** to make sure the picture doesn't come out grainy as well as allowing you to decode the signal properly even if it is weak.



### Frequency reference

As of 26.12.2023, the frequencies these satellites broadcast in are as follows:

|Satellite|Frequency|
|---|---|
|NOAA 15|137.62 MHz|
|NOAA 18|137.9125 MHz|
|NOAA 19|137.1 MHz|
|Meteor M2-3|137.9 MHz|



### Hardware needed to receive these satellites

You will need an SDR and an antenna, no other special equipment is required for VHF.
The SDR should be a reputable brand if you want optimal performance (E.g. AirSpy, RTLSDR Blog, Nooelec...). In simpler terms; avoid aliexpress chinesium sdrs.

As for the antenna, you have the choice between:

##### 1. A simple V dipole antenna
It is incredibly easy to build at the cost of having some nulls and being mor edirectional due to an inconsistent polarization. Starting out with it is a solid idea.

To build it: Get a chock block, stick the shielding of a coax cable in one hole and the copper feed into the other (keep it as short as possible), cut two wires to 54.5 cm and stick them into the holes, spreading them 120° apart in a V shape. That's it. Really.

##### 2. A more permanent QFH antenna
A QFH (Quadrifilar Helix) antenna is much harder to build but performs much better thanks to its true circular polarization. It is the best choice for a more permanent fixture.
I will not be describing how to build it, since the building process is quite involved.


### Software needed

As for software, the by far best choice is:
- [Sdr++](https://github.com/AlexandreRouma/SDRPlusPlus/releases) for recording
- [SatDump](https://github.com/SatDump/SatDump/releases) for decoding. 
Download the nightly builds for both of these, since the tools are relatively new and are being actively developed. There are other chioces but I won't focus on them for the sake of keeping this simple.

For tracking the satellites and figuring out when they pass (They are orbiting the Earth after all), you can use these:

- [Orbitron](http://www.stoff.pl/) - Windows - Dated but functionally sound
- [Gpredict](https://oz9aec.dk/gpredict/) - Windows, Linux, MacOS - A more recent tool, preferred in most cases
- Look4Sat - Android - I personally just use this app, since it provides everything that is needed.


### Actually receiving the satellites!

1. Get to a place with a good view of the sky - The more you can see, the longer you can receive the satellite for and the longer the resulting image will be
2. Configure SDR++ as follows:

#### FOR NOAA APT
- Move your radio to the correct frequency (refer to the frequency chart provided above)
- Rise gain until your noise floor starts to rise
- Select NFM with a bandwidth of about 45000 Hz
- Tick `IF noise reduction` and slect `NOAA APT`
- Turn the low pass filter on, disable the high pass one if it was enabled
- Choose `Audio` as the recording type and once the sat goes into view hit `Record`

#### FOR METEOR-M LRPT
- Go to module manager, add a `Meteor demodulator`
- Turn it on, tick `OQPSK`
- Move it to the correct frequency (As of writing this article, 137.9 MHz)
- Hit `Start` n the demodulator tab

3. Move your V dipole to be hrizontal with the ground and pointed at magnetic north/south, hold it above half a meter off of the floor - my personal advice is to move it around till the noise floor is at its lowest

If everything is right, you are now receiving a beeping APT signal or you see four dots on the demodulator if it is LRPT! 

4. Once the pass is over, hit `stop` on the recorder or demodulator
5. Open SatDump, in the `Offline processing` tab select:
- `Meteor-M2-x 72k LRPT` and `soft` as the file type, put the .s file in the input (NOTE: The symbol rate [72k] might be incorrect depending on the current satellite status, as of writing this guide the correct settings is 72k)
- `NOAA APT` and `audio_wav` as the file type, put the recording in the input

6. Hit `Start`, satdump will now demodulate/decode the input file, it'll plop you into the viewer once it is done
7. You are done! Feel free to play around with the enhancements/settings, you can figure it out :)


## L-band HRPT/HRIT reception

TODO
