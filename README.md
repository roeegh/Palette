<div style="padding-right: 20vw; padding-left: 20vw;">

# How Palette was made?

If you are reading this you probably already know what Palette is. This is a little write-up/blog of my process and instructions for any developers that want to integrate this feature into their MDC/KFD app. If you do end up using this method, crediting me is optional (I will appreciate it tho) but I would at least encourage you to credit [@timac](https://mastodon.social/@timac). This discovery would not be possible without his blog about [Reverse engineering the .car file format](https://blog.timac.org/2018/1018-reverse-engineering-the-car-file-format/).


<u><h3>TLDR</h3></u>
If you only care about how to implement this into your app, go into the `Parsed Files` folder and change 8 bytes of your choice of variable beginning from the `startingOffset` to any color in the BGRA format.
<br>

## Files

While exploring the file system using Filza, I came across a certain set of `.car` files inside `/System/Library/PrivateFrameworks/CoreUI.framework/DesignLibrary-iOS.bundle`. There were a total of 6 files, 3 for light mode and 3 for dark mode.

- `LightStandard.car`
- `LightVibrantStandard.car`
- `LightIncreasedContrast.car`

- `DarkStandard.car`
- `DarkVibrantStandard.car`
- `DarkIncreasedContrast.car`

<u><h3>What's so special about these files?</h3></u>

My first instinct was to open it using Filza but it gave me the "Could not open file" error. Then tyler1029 from the misaka server suggested using [Samra](https://github.com/SerenaKit/Samra) but there were no images, no pdfs, nothing... just an empty file.

At this point, tyler1029 pointed out him seeing `systemBlueColor` in the file when he opened it with a text editor. This leads me to do some research on `.car `files, which lead me to Alexandre's article about [reverse engineering the .car file format](https://blog.timac.org/2018/1018-reverse-engineering-the-car-file-format/). Read the article if you are interested but the spare you the details, Alexandre had developed a tool, CARParser, that provides some much-needed information. 

<u><h3>Parsing the files</h3></u>

When trying to build the CARParser, I had some issues come up which [Nightwind](https://twitter.com/NightwindDev) was kind enough to fix for me and provide a working binary. 

Using the new binary in the following command saves the output into a `.txt` file:

```sh
./CARParser LightStandard.car &> LightStandard.txt
```
<br>

After opening the `.txt` file, my attention went straight to the words `Tree 'COLORS'`. Taking a quick look at the data presented, the most noticeable thing is hexadecimal values and you know what else uses hexadecimal values? COLORS!

```
Tree 'COLORS'
     Key '{length = 132, bytes = 0x00000000 61637469 76697479 426c7565 ... 00000000 00000000 }' -> {length = 12, bytes = 0x010000000000000000ff00ff}
     Key '{length = 132, bytes = 0x00000000 6c616265 6c436f6c 6f720000 ... 00000000 00000000 }' -> {length = 12, bytes = 0x0100000000000000000000ff}
     ...
     Key '{length = 132, bytes = 0x00000000 776f726b 6f757442 6c756543 ... 00000000 00000000 }' -> {length = 12, bytes = 0x0100000000000000ff00bcff}
```
This made me realize that opening the file in a text editor was the wrong choice because the majority of the file was made up of unreadable characters and then it clicked. I had to open the file in a Hex editor!
<br>

The first thing I did after opening the file using [Hex Fiend](https://hexfiend.com/) was to search for any matching hex values and by pure coincidence I found `systemBlueColor`!

![](</Images/systemColorBlueHex.png>)

Now I already knew what I had to look for, searching for the corresponding value, `0100000000000000ff7a00ff` and there it was, just a little bit farther in the file

![](</Images/systemColorBlueHexColor.png>)

## Colors

If you have ever worked with colors will instantly see that the last 4 bytes of that sequence is a hex color code. But most people have only seen hex colors as 6 digits, but here we have 8. The last 2 digits represent alpha/opacity, this is the RGBA format.

<u><h3>I can see colors!</h3></u>

If you enter the hex value for `systemBlueColor`, `#ff7a00`, while ignoring the last 2 digits, you get orange??

![](</Images/systemColorBlueImposter.png>)

Apple, in typical Apple fashion, decided to not use the RGBA format but the BGRA format... Now if we use the BGRA format, we get `#007bff and now we finally see the blue we were looking for!

![](</Images/systemColorBlue.png>)

Now just do this for every color value in all 6 files •ᴗ•


Just kidding...

To save you the trouble, I have parsed and formatted the data from each of the files and placed them with the respective names in the `Parsed Files` folder. There you will see that each file is an array of JSON objects with the following format:

```json
{
    "keyName": "systemBlueColor", // name of the variable
    "hexColorValue": "ff7a00ff",  // original hex color value
    "startingOffset": "17728",    // byte offset to start replacing from
}
```

<u><h3>systemBlueColor lives no more</h3></u>

As a developer, all you have to do is change the 4 bytes starting at the `startingOffset` with the value from the `ColorPicker` UI element. I have not needed to take any extra actions to reduce the file size even when entering `#ffffffff`, which is white.

However, this is only possible on iOS 16 and maybe 17 (if KFD decides to work on the beta). The same files do not exist on iOS 15. I'm however looking to see if I can find the corresponding files, any progress will be shared on my Twitter.

<u><h3>So many colors</h3></u>

What do all of these variables change? What happens if I change `activityColorGreen`? Why are there so many?

I don't know ._.

I'll let you figure that out •ᴗ•


## Contact

The best way to reach me is via Discord or Twitter, @roeegh on both. If you decide to use Discord, please ping me on a server we share so I can look through my message requests otherwise I tend to forget to go through them.
</div>